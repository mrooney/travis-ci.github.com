---
title: GUI & Headless browser testing on travis-ci.org
layout: en
permalink: gui-and-headless-browsers/
---

### What This Guide Covers

This guide covers headless GUI & browser testing using tools provided by the Travis [CI environment](/docs/user/ci-environment/). Most of the content is technology-neutral and does not cover all the details of specific testing tools (like Poltergeist or Capybara). We recommend you start with the [Getting Started](/docs/user/getting-started/) and [Build Configuration](/docs/user/build-configuration/) guides before reading this one.

## Using xvfb to Run Tests That Require GUI (e.g. a Web browser)

You can run test suites that require GUI (like a Web browser) on Travis CI. The environment has `xvfb` (X Virtual Framebuffer) and Firefox installed. Roughly speaking, `xvfb` imitates a monitor and lets you run a real GUI application or Web browser on a headless machine, as if a proper display were attached.

To use xvfb, simply prefix your script with the xvfb wrapper `xvfb-run`,
passing any arguments as you would normally. Here are two examples:

    script:
      - xvfb-run phantomjs mytests.js
      - xvfb-run firefox http://localhost:8080/mytests.html

Under the hood, xvfb-run starts an Xvfb server and runs your command on that display.
When your command finishes, the Xvfb server will be terminated and
xvfb-run will exit with the status of your command.

## Starting a Web Server

If your project requires a Web application running to be tested, you need to start one before running tests. It is common to use Ruby, Node.js and JVM-based Web servers
that serve HTML pages used to run test suites. Because every travis-ci.org VM provides at least one version of Ruby, Node.js and OpenJDK, you can rely on one of those
3 options.

Add a `before_script` to start a server, for example:

    before_script:
      - rackup  # start a Web server
      - sleep 3 # give Web server some time to bind to sockets, etc

If you need Web server to be listening on port 80, remember to use `sudo` (Linux will not allow non-privileged process to bind to port 80). For ports > 1024, using `sudo`
is not necessary (and not recommended).



## Using Phantom.js

[Phantom.js](http://www.phantomjs.org/) is a headless WebKit with JavaScript API. It is an optimal solution for fast headless testing, site scraping, pages capture, SVG renderer, network monitoring and many other use cases.

[CI environment](/docs/user/ci-environment/) provides Phantom.js preinstalled at `/usr/local/bin/phantomjs`. You'll need to use xvfb-run (see the section above).

## Examples

### Real World Projects

 * [Ember.js](https://github.com/emberjs/ember.js/blob/master/.travis.yml) (starts Web server programmatically)


### Ruby

#### RSpec, Jasmine, Cucumber

Here's an example rake task that runs Rspec, Jasmine, and Cucumber tests:

    task :travis do
      ["rspec spec", "rake jasmine:ci", "rake cucumber"].each do |cmd|
        puts "Starting to run #{cmd}..."
        system("export DISPLAY=:99.0 && bundle exec #{cmd}")
        raise "#{cmd} failed!" unless $?.exitstatus == 0
      end
    end

In this example, both Jasmine and Cucumber need the display port, because they both use real browsers. Rspec would run without it, but it does no harm to set it.
