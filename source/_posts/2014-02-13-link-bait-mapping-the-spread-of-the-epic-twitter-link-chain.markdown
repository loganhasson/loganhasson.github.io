---
layout: post
title: "Link Bait: Mapping the Spread of the Epic Twitter Link Chain"
date: 2014-02-13 21:29
comments: true
categories: bfs, algorithms, twitter, viral, ruby
---

![Map of Twitter Link Chain](http://i460.photobucket.com/albums/qq324/lhvio89/feb_14_small_png.png)

### TL;DR

I used Breadth-first Search to map the link chain that's currently making its way around twitter. So far, the data shows:

1. Clustering coefficient: 0.003
2. Characteristic Path Length: 18.252
3. Average Number of Neighbors: 2.023

Code available here: [Fork and Pull Request Away!](https://github.com/loganhasson/twitter-link-chain)

Oh, and for the record, it all started [here](https://twitter.com/BenHowe/statuses/433249756006346752) with a link to a 404 page.

### Update: Feb 14, 7:30am

The graph has a ton of new nodes now. The stats are pretty similar, though the characteristic length is abou 17 now, and the furthest degree of separation is about 44. If you'd like to play around with a high-res version of that image above, check it out [here](https://www.dropbox.com/s/soz79f1nnccnrds/feb_14_pdf.pdf).

# Intro

Like many of you, the past few days my Twitter feed was inundated with several tweets claiming to link to something so shocking and controversial I had to see it *right now*. I'm not usually one to fall for obvious link bait, but I gave one a go.

And then I clicked on the link in the status I was led to.

And I clicked on the link there.

And then I caught on. I'd fallen for Twitter's newest bit of viral fun: the link chain.

# What is this link chain?

If you haven't see it, there are several links going around that send you on a wild goose chase through the world of Twitter. Somebody posts a link, claiming its points to some extremely shocking story, but it really just links to another twitter status. That status, then, links to another. And so on and so forth.

And the prize at the end of the tunnel? A 404 error. Nice.

I didn't end up jumping in and spreading the link forward, but it did get me thinking: what must this massive link chain look like? How does something like this get started, and how far does it spread?

Enter Ruby and Breadth-first Search.

# Mapping the Chain: The Problem

I'd played around with [Breadth-first Search](http://en.wikipedia.org/wiki/Breadth-first_search) once or twice before, and I figured this problem felt like a great use case for the algorithm. If you think about this chain of tweets as a series of nodes and neighbors, it breaks down somewhat like this:

1. Take any random tweet that has one of these links in it
2. It's neighbors are any tweets that, in turn, link back to it

Seems pretty straight forward, right?

Not so fast. There were two primary problems that arose quite quickly:

1. How in the heck do I find the tweets that link back to a particular status?
2. A Breadth-first search (from here on out, to be referred to as BFS) sort of assumes you are starting at the "beginning" of something and expanding from there. I was actually starting somewhere in the middle of this chain.

The first problem turned out to be pretty easy to solve, after realizing one particular quirk of the Twitter search API.

# Finding the Neighbors

Since every tweet that links back to a particular status, by definition, contains that status' link as some text, I could just do a search for the tweet of interest's URL.

There was only one problem with that: the Twitter search API is case sensitive, while usernames *aren't*. Therefore, one tweet's status could be linked to in any number of ways.

The solution? Search for the tweet's ID. Duh. Sadly, that took my a couple of hours to figure out.

# Going Back to the Beginning

The other problem took a bit more time to figure out. It turns out, though, that the key to it was the realization that it was, in fact, pretty simple to figure out where the beginning of the whole link chain was.

I just had to write a simple bit of code to get back to the beginning from any tweet in the chain:

```ruby
def self.get_first_tweet_hash(starting_status_url)
  id = starting_status_url[/(\d)+$/].to_i
  link = true
  while link
    begin
      tweet = CLIENT_ONE.status(id)
      puts "#{tweet.user.screen_name}: #{id}"
      if tweet.urls
        id = tweet.urls.first.expanded_url.to_s[/(\d)+$/].to_i
      else
        link = false
      end
    rescue
      link = false
    end
  end

  {
    :username => tweet.user.screen_name,
    :id => tweet.id,
    :created_at => tweet.created_at,
    :retweet => tweet.retweet?,
    :retweet_count => tweet.retweet_count,
    :location => nil
  }

end
```

(Note: `CLIENT_ONE` is my interface to the [Twitter Gem](https://github.com/sferik/twitter).)

Essentially, I look at a status, grab the link of that status, and go look at it. I do this until I find no more links. And then I have the beginning. I had to throw in the rather ugly `begin rescue` block because the last tweet happens to link to a `404`.

Once I had the starting tweet, it was just a matter of starting the search.

# BFS

I'll let you read up on BFS, but here's the code that actually runs the search:

```ruby
def map_graph
  while !tweet_queue.empty?
    tweet = tweet_queue.shift
    tweet.get_neighbors.each do |neighbor|
      if !visited?(neighbor)
        add_to_path(tweet, neighbor)
        add_to_arrays(neighbor)
      end
    end
  end
end
```

In simple terms, you take a node and throw it into a queue. Then, you find all of its neighbors and throw them into a queue. Then, you take the first node from the queue and repeat. In this way, you traverse a graph, level by level, until you reach some pre-determined end point, or until there are no more nodes.

# Plotting the Graph

I discovered the *amazing* [Graph Gem](https://github.com/seattlerb/graph) which makes plotting directional graphs insanely simple. You get a `digraph` method that accepts a block, in which you define edges. At the end of the method, you can say something like `save "file", "png"` and you get a nice bit of output. You are also left with useful data in `.dot` file format, which is easily imported into other visualization software. (I happened to use Cytoscape for the graph at the top of this post.)

Here's the code to plot the data:

```ruby
def display_graph(traveled_path)
  digraph do
    traveled_path.each do |pair|
      start = pair[0] == nil
      if start
        edge "#{pair[1].username}: #{pair[1].created_at.strftime("%a %b %e - %k:%M")}", "Start"
      else
        edge "#{pair[1].username}: #{pair[1].created_at.strftime("%a %b %e - %k:%M")}", "#{pair[0].username}: #{pair[0].created_at.strftime("%a %b %e - %k:%M")}"
      end
    end

    save "test", "png"
  end
end
```

What makes this especially easy is the fact that the BFS algorithm leaves me with an array that contains the entire history of the graph in parent - child pairs. All it takes is a quick iteration through that array, and you have a fancy graph.

# Considerations

There were a couple of logical considerations that had to be made in the course of running this search.

The biggest problem was dealing with the Twitter search API. It happens to only return about 100 results per search, and there isn't any really good way to figure out the next page of results. You aren't immediately made aware of the fact that there *is* a next page of results even. And without checking all results, you are bound to miss large portions of the graph.

To deal with this, I accessed a private method in the `Twitter::SearchResults` class of the Twitter Gem (linked to above). This rather sweet method returns the correct options for a search that would return the next page of results. So, in the `get_neighbors` method, I needed to keep making the same search (for a given status id) until there were no more pages of results.

The other problem was dealing with people who decided to throw recursive loops into the mix. Several users would jump into the chain, and then link to another status of their own. This created mini loops. Easy solve: reject any neighbors that have the same username as the current tweet.

# Current Issues

There are still some problems to solve. The most major of which is the rate limiting that Twitter has on its search API. As it stands, it is about 180 requests per 15 minute block. For a graph of this size, that's not nearly enough.

The other problem, which I've created a stop-gap for, is the issue of how to store progress if graph traversal has to go in spurts. Using `YAML::Store`, I keep track of the current node and the current state of all queues. This seems to work, but I fear that tweets get lost in between traversals.

# Future Plans

I'm hoping to graph all of this data in much more meaningful ways. Since I'm collecting location data (for tweets that have it) and temporal data, it'd be really awesome to create an interactive, geo-located graph of this link chain spreading over time and distance. Furthermore, I am collecting retweet information, and am hoping to find meaningful ways to display that.

# Help!

This is a pretty large map, and it's pretty difficult to traverse it 180 requests at a time, running a cron job every 20 minutes. Because the data is being stored in YAML format, and the edges are stored in simple node pairs in a `.dot` file, it'd be pretty easy to distribute the graphing of this. Merging multiple `.dot` files would be rather trivial. If the larger open source community could jump on this and start graphing from different points in the map, we could make quick work of it. Fork the project on [GitHub](http://github.com/loganhasson/twitter-link-chain)!

# More Pretty Pictures

Full sized images can be found [here](https://www.dropbox.com/sh/izf9dt28ys0hq6y/yTeVh9G8Nn). They are all pretty similar, but give a decently different view of the graph that I decided to post them all. There is also a PDF which allows for much more exploration of the graph.

# Nerdy Graphs

![Neighborhood Connectivity](http://i460.photobucket.com/albums/qq324/lhvio89/neighborhood-connectivity.png)

![In-Degree Distribution](http://i460.photobucket.com/albums/qq324/lhvio89/in-degree-distribution-1.png)

![Average Clustering Coefficient Distribution](http://i460.photobucket.com/albums/qq324/lhvio89/average-clustering-coef-distribution-1.png)

![Shortest Path Length Distribution](http://i460.photobucket.com/albums/qq324/lhvio89/shortest-path-length-distribution.png)