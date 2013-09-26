---
layout: post
title: "bash trash"
date: 2013-09-25 16:48
comments: true
sharing: true
footer: false
author: Logan Hasson
---
This isn't too terribly exciting, but I've been on a big "write-everything-in-fact-*all-the-things*-as-functions-for-bash" kick these last couple of days, and I am pretty happy with my latest one. Though I rarely find myself in the position of having files in my Trash any more, I still occasionally drag something in there from my downloads folder (since they are right next door to one another on my dock). I hate, hate, hate, hate, hate having a cluttered Trash icon, so I empty it all the time.

While it's totally possible to just press `Shift+Command+Delete` while in Finder, I'd much rather not have to switch out of terimal if possible. So, I wrote this little function and threw it in my .bash_profile. Now, whenever I want to empty the trash, I just have to type `empty`. It's lovely. And as a bonus, I even get to see how many files were deleted.

```
function empty () {
  pushd ~/.Trash > /dev/null
  tmp=$(rm -rfv * | wc -l | sed -e 's/^[ \t]*//')
  if [ $tmp == "1" ]; then
    echo "$tmp file was removed."
  else
    echo "$tmp files were removed."
  fi
  pushd > /dev/null
}
```

Here's a commented version if you're interested. (Note, I looked up the regular expression for one of the commands. Regular expressions are hard.)

```
# Commented Version
# USE: empty
 
function empty () {
  # push the CWD into the stack and change to the ~/.Trash directory. Pipe this output to /dev/null so it doesn't print to
  # the console

  pushd ~/.Trash > /dev/null

  # assign a variable, tmp, to the following
  # rm -rfv * deletes everything in the trash. the -v flag tells it to echo each file as it deletes
  # this is then piped to wc -l, which counts the lines that were outputted
  # that count gets a bunch of whitespace put in front of it for some reason, and is piped to the stream editor (sed), and the
  # -e flag lets us pass a fancy regex to strip the leading whitespace

  tmp=$(rm -rfv * | wc -l | sed -e 's/^[ \t]*//')

  # check to see if $tmp is 1, if so we want to echo singular "file"

  if [ $tmp == "1" ]; then
    echo "$tmp file was removed."
  else

  # otherwise echo plural "files"

    echo "$tmp files were removed."
  fi

  # pushd again brings us back to the directory we were in before running the command, and the output is again piped to
  # /dev/null

  pushd > /dev/null
}
```
