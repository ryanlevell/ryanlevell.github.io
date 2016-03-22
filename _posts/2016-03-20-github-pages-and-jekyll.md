---
layout: post
title:  "GitHub Pages and Jekyll Setup On A Mac"
date:   2016-03-20 21:33:03 -0500
categories: github-pages jekyll mac
---

<hr>
### Setting up GitHub pages for a user

These steps can be found at [GitHub Pages](https://pages.github.com/):

Create a new repo at GitHub.  
* Repo name must match exactly: `<username>.github.io`  
  
Clone the repo
{% highlight bash linenos %}
~ $: git clone https://github.com/<username>/<username>.github.io
{% endhighlight %}
Add an index.html
{% highlight bash linenos %}
~ $: cd <username>.github.io
~ $: echo "test" > index.html
{% endhighlight %}
Push your changes
{% highlight bash linenos %}
~ $: git add --all
~ $: git commit -m "first commit"
~ $: git push -u origin master
{% endhighlight %}
View your web site  
* `http://<username>.github.io`

<hr>
### Setting up Jekyll

The original steps as found at the [Jekyll Website](https://jekyllrb.com/docs/quickstart/):

{% highlight bash linenos %}
~ $: gem install jekyll
~ $: jekyll new <name of blog>
~ $: cd <name of blog>
~ $: jekyll serve
{% endhighlight %}
(Blog can be viewed at `localhost:4000`)

<hr>
### Setting up Jekyll (My Mac experience)

After installing Jekyll successfully, I could not use the command:
{% highlight bash linenos %}
~ $: gem install jekyll
~ $: jekyll new <name of blog>
-bash: jekyll: command not found
{% endhighlight %}

Below solutions found at [StackOverflow](http://stackoverflow.com/questions/8146249).

#### Possible solution #1: Add the gem environment paths to `$PATH` in `.bash_profile`.

{% highlight bash linenos %}
~ $: gem env
RubyGems Environment:
  # Additonal lines removed
  - GEM PATHS:
     - /Users/ryan/.rbenv/versions/2.0.0-p481/lib/ruby/gems/2.0.0
     - /Users/ryan/.gem/ruby/2.0.0
  # Additonal lines removed
~ $: echo 'PATH=$PATH:<path to gem>' >> ~/.bash_profile
{% endhighlight %}

#### Solution #2: That I used

{% highlight bash linenos %}
~ $: rbenv rehash
~ $: # Run the original commands again
{% endhighlight %}

which is described below:

{% highlight bash linenos %}
~ $: rbenv help rehash
Usage: rbenv rehash
Rehash rbenv shims (run this after installing executables)
{% endhighlight %}

<hr>
#### NOTES
1. GitHub Pages are powered by Jekyll behind the scenes.
2. The Jekyll files must be at the root folder of the GitHub repo.
 * `jekyll new .`
3. There is a way to put the Jekyll blog on a subfolder of your repo:
 * Create a separate repo with Jekyll as the root and link it to your current page.
 * * This new repo would be a project so the `gh-pages` branch would have to be used for it.
 * Create new `_layouts` and `_includes` and permalinks to move the blog to subfolder.
 * * I actually began doing this, then reverted it all as I decided not to have a custom, non-blog homepage.
4. You may see people using the branch `gh-pages`. This is only needed when creating a GitHub page for a project repo. This post was for creating a GitHub page for a user, so the `master` branch is fine.
5. I already had Ruby installed from previous side work, but I am guessing I installed with homebrew:
{% highlight bash linenos %}
~ $: brew update
~ $: brew install rbenv
~ $: brew ruby-build
~ $: rbenv install <version number> # newest at time of post was 2.3.0
~ $: rbenv local <version number>
~ $: rbenv rehash
{% endhighlight %}
