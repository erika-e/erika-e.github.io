---
title: 'Analytics Engineering Dotfiles'
date: 2021-11-29
description: Automating configuration of an M1 Macbook for analytics engineering. Includes configuration of VSCode for working with dbt and Jinja. Fork it for easy setup of your own mac, or check it out for inspiration for zsh aliases.
featured_image: '/images/posts/succulents.jpeg'
---

## What are dotfiles, anyway?

Dotfiles are configuration as code for your development environment. The name
originates from files like `.gitignore` or `.zshrc`, which start with a `.` and
became referred to as dotfiles.

One of the cool things about [dotfiles in the software world](https://dotfiles.github.io/)
is that they're frequently shared. You can learn from what other people have
done. I was introduced to dotfiles as a concept by a software engineering
colleague of mine.

There don't seem to be many examples of analytics engineering dotfiles out
there. What I did find is usually called "setup" and not dotfiles[^3].

Here are some examples:

* [Canonical Claire Carroll - How we Set Up Our Computers for Working with dbt](https://discourse.getdbt.com/t/how-we-set-up-our-computers-for-working-on-dbt-projects/243)
* [Anders on VSCode setup for the dbt CLI](https://discourse.getdbt.com/t/setting-up-vscode-to-use-with-the-dbt-cli/3291)
* [GitLab's Onboarding Script](https://gitlab.com/gitlab-data/analytics/-/blob/master/admin/onboarding_script.zsh)

## To Fork or Not to Fork

It turns out there's a whole debate on [whether](https://zachholman.com/2010/08/dotfiles-are-meant-to-be-forked/)
or [not](https://www.anishathalye.com/2014/08/03/managing-your-dotfiles/) you should
fork someone else's dotfiles. Forking in this context means copying someone
else's approach for your own environment.

If you're new to working with dbt, my dotfiles might make a good starting point.
However, dotfiles **are** highly personal. What works for me might not work for
you. Even if my exact setup is perfect for you, you'd need to learn what I know
to get the same efficiency I do. Your pain points might not be the same as mine,
and things I love might not work for you at all.

With that said, please do **fork away**! I tried to do a good job documenting
what's going on in comments and on the repo. I'd love to see pull requests, issues,
and contributions to this repo. We're still in the early days of analytics
engineering as a discipline. I don't think we can share enough.

I'm just getting started with customizing my command line environment. I'm sure
there's a lot that could be improved here. Like [Stone Soup](https://en.wikipedia.org/wiki/Stone_Soup),
I'm hoping that by putting this out in the world, people will be moved to
show up and put something in the pot.

[Erika's analytics engineering dotfiles on GitHub](https://github.com/erika-e/dotfiles)

## A Little Bit of Erika by Your Side[^Mambo]

Here's some of the things the dotfiles do that I think are pretty neat.

* 🍺  Manage installed software and packages with Homebrew
* 🦾  Optimize VSCode for working with dbt by installing [lots of good extensions](https://github.com/erika-e/dotfiles/tree/main/vscode)
* 👩‍🔬  Clone a dbt project so you can easily test your setup on a new computer
* ✨  Add aliases for common dbt CLI operations
* 🔎  Make validating data for dbt PRs easy with dbt-audit-helper automation
* 🤘  Make adding to your dbt project easier with shell script customization for dbt-codegen
* 🧹  Basic SQLfluff setup for dbt to make linting SQL easy

## Making the Dotfiles

I wanted the dotfiles to be simple and approachable. By approachable, I mean
that I wanted them to be easily understandable to someone with an analytics
background[^1] and solid dbt experience.

I balanced my desire to have this be general and useful with the need to keep my
own configurations centralized. I chose tools that are free to use. For now,
I've chosen not to bring in a dotfile utility.

I kept things simple, until I couldn't. For example, I started with all the files
in one root directory. I thought people might have trouble figuring out what was
what, so I added some documentation. It still seemed confusing, so I added
subdirectories. That inspired me to add READMEs for each tool's configs.

I'm sure things will continue to evolve. There are some tools I've used, like
Terraform, that I'm not currently working with and didn't include for that reason.

## About Configuring Your Shell

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">I found shell configuration intimidating and procrastinated on it for a long time even though I use the command line a lot.<br><br>It&#39;s amazing to have automation for common tasks, even stuff that saves a few keystrokes adds up throughout the day.</p>&mdash; Erika Pullum (Swartz) she/hers (@erikapullum) <a href="https://twitter.com/erikapullum/status/1453814316071854084?ref_src=twsrc%5Etfw">October 28, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Shell configuration has been life changing for my daily workflows. I wish I'd done
it sooner. In the course of building the dotfiles shell scripts I learned some
nifty new command line tools.

I highly recommend investing time in learning the ins and outs of the tools you
use every day. A little bit goes a long way.

[^3]: Sometimes it feels like tooling and practices in the data world are a few
    years behind the software engineering world. Some of that is terminology,
    like dotfiles vs. setup. However, the problems data folks are solving are
    different and the data tooling isn't as good as it should be - yet.

[^1]: I realized after I wrote most of this post that some concepts in the
    dotfiles might not be intuitive to the person I'm describing if they haven't
    come across them before. For example, someone who has mostly worked with dbt
    Cloud might not be very familiar with shell customization. Trying to explain
    everything felt like it would overload this post. If there's something you'd
    really like to see me write more about, let me know on Twitter!

[^Mambo]: Yep, that's a Mambo #5 Joke. Hope it's not stuck in your head now.
