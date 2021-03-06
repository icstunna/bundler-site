---
title: FAQ - Frequently Asked Questions
---

## FAQ - Frequently Asked Questions
  
### Why Can't I Just Specify Only <code>=</code> Dependencies?

<strong>Q:</strong> I understand the value of locking my gems down
to specific versions, but why can't I just specify <code>=</code> versions
for all my dependencies in the <code>Gemfile</code> and forget about
the <code>Gemfile.lock</code>?

<strong>A:</strong> Many of your gems will have their own
dependencies, and they are unlikely to specify <code>=</code> dependencies.
Moreover, it is probably unwise for gems to lock down all of *their*
dependencies so strictly. The <code>Gemfile.lock</code> allows you to
specify the versions of the dependencies that your application needs in
the <code>Gemfile</code>, while remembering all of the exact versions of
third-party code that your application used when it last worked correctly.

By specifying looser dependencies in your <code>Gemfile</code>
(such as <code>nokogiri ~> 1.4.2</code>), you gain the ability to run
<code>bundle update nokogiri</code>, and let bundler handle updating **only**
<code>nokogiri</code> and its dependencies to the latest version that still
satisfied the <code>~> 1.4.2</code> version requirement. This also allows you
to say "I want to use the current version of nokogiri" (<code>gem 'nokogiri'</code>
in your <code>Gemfile</code>) without having to look up the exact version number,
while still getting the benefits of ensuring that your application always runs with
exactly the same versions of all third-party code.

### Why Can't I Just Submodule Everything?

<strong>Q:</strong> I don't understand why I need bundler to manage
my gems in this manner. Why can't I just get the gems I need and stick them
in submodules, then put each of the submodules on the load path?

<strong>A:</strong> Unfortunately, that solution requires that you
manually resolve all of the dependencies in your application, including dependencies
of dependencies. And even once you do that successfully, you would need to redo that
work if you wanted to update a particular gem. For instance, if you wanted to update
the <code>rails</code> gem, you would need to find all of the gems that depended on
dependencies of Rails (<code>rack</code>, <code>erubis</code>, <code>i18n</code>,
<code>tzinfo</code>, etc.), and find new versions that satisfy the new versions of
Rails' requirements.

Frankly, this is the sort of problem that computers are good at, and which you,
a developer, should not need to spend time doing.

More concerningly, if you made a mistake in the manual dependency resolution
process, you would not get any feedback about conflicts between different dependencies,
resulting in subtle runtime errors. For instance, if you accidentally stuck the wrong
version of <code>rack</code> in a submodule, it would likely break at runtime, when
Rails or another dependency tried to rely on a method that was not present.

<strong>Bottom line:</strong> even though it might seem simpler at first glance, it is decidedly significantly
more complex.

### Why Is Bundler Downloading Gems From <code>--without</code> Groups?

<strong>Q:</strong> I ran <code>bundle install --without production</code> and
bundler is still downloading the gems in the <code>:production</code> group. Why?

<strong>A:</strong> Bundler's <code>Gemfile.lock</code> has to contain exact
versions of all dependencies in your <code>Gemfile</code>, regardless of any options
you pass in. If it did not, deploying your application to production might change all
your dependencies, eliminating the benefit of Bundler. You could no longer be sure that
your application uses the same gems in production that you used to develop and test with.
Additionally, adding a dependency in production might result in an application that is
impossible to deploy.

For instance, imagine you have a production-only gem (let's call it
<code>rack-debugging</code>) that depends on <code>rack =1.1</code>. If we did not evaluate
the production group when you ran <code>bundle install --without production</code>, you
would deploy your application, only to receive an error that <code>rack-debugging</code>
conflicted with <code>rails </code> (which depends on <code>actionpack</code>, which depends
on <code>rack ~> 1.2.1</code>).

Another example: imagine a simple Rack application that has <code>gem 'rack'</code>
in the <code>Gemfile</code>. Again, imagine that you put <code>rack-debugging</code> in the
<code>:production</code> group. If we did not evaluate the <code>:production</code> group when
you installed via <code>bundle install --without production</code>, your app would use
<code>rack 1.2.1</code> in development, and you would learn, at deployment time, that
<code>rack-debugging</code> conflicts with the version of Rack that you tested with.

In contrast, by evaluating the gems in **all** groups when you call <code>bundle install</code>,
regardless of the groups you actually want to use in that environment, we will discover the
<code>rack-debugger</code> requirement, and install <code>rack 1.1</code>, which is also compatible
with the <code>gem 'rack'</code> requirement in your <code>Gemfile</code>.

<strong>In short,</strong>
by always evaluating all of the dependencies in your Gemfile, regardless of the dependencies
you intend to use in a particular environment, you avoid nasty surprises when switching to a different
set of groups in a different environment. And because we just download (but do not install) the gems,
you won't have to worry about the possibility of a difficult **installation** process for a gem that
you only use in production (or in development).

### I Have a C Extension That Requires Special Flags to Install

<strong>Q</strong>: I have a C extension gem, such as <code>mysql</code>, which requires
special flags in order to compile and install. How can I pass these flags into the installation
process for those gems?

<strong>A</strong>: First of all, this problem does not exist for the <code>mysql2</code>
gem, which is a drop-in replacement for the <code>mysql</code> gem. In general, modern C extensions
properly discover the needed headers.

If you really need to pass flags to a C extension, you can use the <code>bundle config</code>
command:

    $ bundle config build.mysql --with-mysql-config=/usr/local/mysql/bin/mysql_config

Bundler will store this configuration in <code>~/.bundle/config</code>, and bundler will use
the configuration for any <code>bundle install</code> performed by the same user. As a result, once
you specify the necessary build flags for a gem, you can successfully install that gem as many times
as necessary.

### I Do Not Have an Internet Connection and Bundler Keeps Trying to Connect to the Gem Server

<strong>Q</strong>:  I do not have an internet connection but I have installed the gem before.
How do I get bundler to use my local gem cache and not connect to the gem server?

<strong>A</strong>: Use the --local flag with bundle install. The --local flag tells bundler
to use the local gem cache instead of reaching out to the remote gem server.

    $ bundle install --local

### Bundling From RubyGems is Really Slow

<strong>Q</strong>: When I bundle from rubygems it is really slow. Is there anything I can do to make it faster?

<strong>A</strong>: Add the --full-index flag when bundling from the rubygems server. This downloads
the index all at once instead of making numerous small requests to the api.

    $ bundle install --full-index
