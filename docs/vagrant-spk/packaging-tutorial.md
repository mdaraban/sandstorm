# Packaging tutorial (Generic)

Style: Hands-on introductory tutorial.

This tutorial will show you how to package an app for
[Sandstorm](https://sandstorm.io) in five minutes. (Sometimes the
downloads can take up to 30 minutes, but the interactive time is about
five minutes. Any slow steps are labeled accordingly.) Going through
this tutorial, you'll learn:

* How to take an existing web application and turn it into a Sandstorm
  package (SPK).

* How our packaging helper (`vagrant-spk`) lets you edit the app's files on your main operating
  system (we support Mac OS, Linux, and Windows), even though Sandstorm apps always run on Linux.

The tutorial uses a PHP app as an example. **Sandstorm supports any
programming language that runs on Linux**, not just PHP, such as
Meteor, Python, Rails, Node, PHP, C++, Go, Rust, and more. Read about
[vagrant-spk's platform stacks](platform-stacks.md) to see how to optimize
your package for your app's programming language.

Once you've worked through this tutorial, look in the **Next steps** section
at the bottom of this document to learn more about how to improve the speed
of packaging on your computer, learn about user authentication and
permissions in Sandstorm, and more.

## Overview of Sandstorm packages

A Sandstorm application package includes all the code needed to run your
app, including all binaries, libraries, modules, etc. Normally, figuring out
exactly what to put in a package could be tedious, and if you decide to include
everything, it would result in a massive package file. Sandstorm makes it easy to keep things
small by employing a trick: it watches your server running on your development machine
and pulls in all the files the server uses.

Keeping the package small means that it's easy for people to install and update
the package you're creating.

Sandstorm packages rely on the Sandstorm platform to handle adding new user
accounts and other access control elements. Read more in the
[App Developer Handbook](../developing/handbook.md).

## Before proceeding, install vagrant-spk

Make sure you've worked through the
[vagrant-spk installation guide](installation.md) before going through this tutorial!

# Creating a package

Over the course of section, we will use `vagrant-spk` to create a
Sandstorm package containing the app and all its dependencies.

### Choose an app that you will package

This tutorial uses a sample PHP web app. To download it, run the following commands:

```bash
cd ~/projects
git clone git://github.com/paulproteus/php-app-to-package-for-sandstorm
```

The app's code will be stored at
`~/projects/php-app-to-package-for-sandstorm`.  We will spend the rest
of the tutorial in that directory and its sub-directories.

**Note**: Feel free to spend a moment looking around this folder you
just downloaded (or [view its contents on
GitHub](https://github.com/sandstorm-io/php-app-to-package-for-sandstorm)).
You'll find an `index.php` and some CSS and Javascript.

## Create .sandstorm, to store packaging information for the app

Over the course of this tutorial, we will create and modify the
`.sandstorm/` directory in the project. This directory contains the Sandstorm
packaging information, such as the app name and the scripts required to
create the package.

We'll use the `vagrant-spk` tool to create this directory.

The purpose of `vagrant-spk` is to create a Linux system where Sandstorm and
your app run successfully. It acts differently based on which 
[platform stack](https://docs.sandstorm.io/en/latest/vagrant-spk/platform-stacks/)
you want to use. In our case, we'll use the _lemp_ platform: Linux, nginx, MySQL,
and PHP.


To do that, run the following commands:

```bash
cd ~/projects/php-app-to-package-for-sandstorm
vagrant-spk setupvm lemp
```

You should see a message like the following:

```bash
Initializing .sandstorm directory in /Users/myself/projects/php-app-to-package-for-sandstorm/.sandstorm
```

You should also find that the `.sandstorm/` directory now exists in your project.
Here's how you can take a look:

```bash
ls ~/projects/php-app-to-package-for-sandstorm/.sandstorm
```

## Start a virtual Linux machine containing Sandstorm

The next step is to start the virtual Linux machine, containing
Sandstorm, that you will develop the package with. To do that, run the following
command:

```bash
vagrant-spk vm up
```

(You should be running it from the `~/projects/php-app-to-package-for-sandstorm` directory.)

You will see a _lot_ of messages printed out. Some of them are not necessary;
we're working on tidying up the scripts to minimize the noise.

Eventually, you will see this mesage:

```bash
==> default: Processing triggers for php5-fpm (5.6.9+dfsg-0+deb8u1) ...
```

and get your shell back. At this point, you can continue to the next step.

**Troubleshooting note**: If you already have Sandstorm installed on your laptop,
you might see the following red text:

```bash
Vagrant cannot forward the specified ports on this VM, since they
would collide with some other application that is already listening
on these ports. The forwarded port to 6080 is already in use
on the host machine.
```

If you see that, run:

```bash
sudo service sandstorm stop
```

and halt any other `vagrant-spk` virtual machines you might be using to develop
other apps.

## Examine the Sandstorm instance you will develop against

Your system is now running a Sandstorm instance. You should visit it
in your web browser now by visiting

http://local.sandstorm.io:6080/

Take a moment now to sign in by clicking on **Sign in** in the top-right corner.
Choose **Sign in with a Dev account** and choose **Alice (admin)** as the user
to sign in with.

Note that there are other "dev accounts" available -- you can use this to test
the experience of using your app as other users.

Over the next few steps, we will prepare the app so that it is visible in that
Sandstorm instance.

<!--(**Editor's note**: We should make localhost:6080 work, so that people don't have to learn about `local.sandstorm.io`.)-->


## Use vagrant-spk to create a package definition file for your app

Every Sandstorm package needs to declare its name. It does this in a
`sandstorm-pkgdef.capnp` file. (`capnp` is short for Cap'n Proto; for the
purpose of this tutorial, Cap'n Proto is a configuration file format that is
easy for Sandstorm to parse.)

Let's use `vagrant-spk` to create a package definition file by running:

```bash
vagrant-spk init
```

(You should be running it from the `~/projects/php-app-to-package-for-sandstorm` directory.)

This will create a new file called `.sandstorm/sandstorm-pkdef.capnp`.

We'll make two changes. First, we'll give our app a **title** of
_Sandstorm Showcase_. To do that, open `.sandstorm/sandstorm-pkgdef.capnp` in
a text editor and find the line with this text:

```bash
    appTitle = (defaultText = "Example App"),
```

Change it to the following.

```bash
    appTitle = (defaultText = "Sandstorm Showcase"),
```

Second, we will customize the text that Sandstorm users see when they want
to create a new _instance_ of the app. To do this, find the line containing:

```bash
      ( title = (defaultText = "New Instance"),
```

and change it to read:

```bash
      ( title = (defaultText = "New Showcase"),
```

## Make the app available in the Sandstorm, in development mode

Sandstorm app development focuses around `dev` mode, which makes the current
code of an app available.

Make this app available in dev mode by doing:

```bash
vagrant-spk dev
```

(You should be running it from the `~/projects/php-app-to-package-for-sandstorm` directory.)

On the terminal, you will see a message like:

```bash
App is now available from Sandstorm server. Ctrl+C to disconnect.
```

Now you can visit the Sandstorm at http://local.sandstorm.io:6080/ and log in
as **Alice (admin)**. Your app name should appear in the list of apps.

You can click **New Showcase** and see the PHP code running.

Note that each app instance (each "Showcase", for this app) runs separate from
each other. You can see that for this app because the app stores the number
of times you have reloaded the page. If you create another **New Showcase**,
each instance will store their data separately.

In Sandstorm, resources like a database are embedded into the package. That
helps enforce this isolation between app instances.

## Stop the development server and create a package file

The final step in this tutorial is to create a Sandstorm package (SPK) file,
containing the app and all its dependencies. This SPK file is something that
you can give directly to people who want to use the code, or add to the
Sandstorm app list so people can install it easily from their own servers.

To do that, we first stop the `vagrant-spk dev` server. To do that, type
`Ctrl-C` on your keyboard. You will see some messages like:

```bash
Unmounted cleanly.
Updating file list.
```

If you're still logged into your Sandstorm instance, you will notice the app
vanishing from the list of apps on the left.

To create the SPK file, run:

```bash
vagrant-spk pack ~/projects/package.spk
```

(You should be running it from the `~/projects/php-app-to-package-for-sandstorm` directory.)

This will take a few moments, and once it is done, there will be a file in
`~/projects/package.spk` that contains the full app.

You can see how large it is by running the following command:

```bash
du -h ~/projects/package.spk
```

On my system, I see:

```bash
16M
```

or sixteen megabytes.

Now, you can upload this to your development Sandstorm instance by clicking
**Upload app* and choosing this file in your web browser's upload dialog.

To learn how to go further and share this SPK file, or what you should know
for other web frameworks, check out the **What's next** section below.

<!--(**Editor's note**: IMHO vagrant-spk pack should auto-guess a reasonable package filename.)-->

# Stop the virtual machine running your app and Sandstorm

With `vagrant-spk`, before you can develop a second app, you must stop
the virtual machine created as part of developing the first one.  This
is because the `vagrant-spk` virtual machine always uses port 6080.

In our case, we're done using the virtual machine running this app, so
it's safe to stop it. Run this command:

```bash
vagrant-spk vm halt
```

(You should be running it from the `~/projects/php-app-to-package-for-sandstorm` directory.)

Now port 6080 is available for other app packaging projects. If you ever want to work on
this app's packaging again, you can bring it up by running:

```bash
vagrant-spk vm up
```

If you ever are confused about which Vagrant virtual machines are
running, you can try this command:

```bash
vagrant global-status
```

(**Note**: It's `vagrant` here, not `vagrant-spk`.)

# What's next

Now that you've seen the basics of how a Sandstorm app works, you
might be interested in any of the following:

* What makes a great Sandstorm app? See the [App Developer Handbook](../developing/handbook.md).
* How do I learn more about the technical underpinnings of `vagrant-spk`? How do I make `vagrant-spk` faster?
Read about [understanding & customizing vagrant-spk](customizing.md).
* How do I package-up a Python, Meteor, or other non-PHP app? Read about [platform stacks](platform-stacks.md).
* Will this work on Windows? Yes, but you might experience issues with symbolic links or long filenames. There are often known workarounds for these issues, so if you run into them, please [request help on the development group](https://groups.google.com/forum/#!forum/sandstorm-dev) or IRC.
* Will this work if I typically run Sandstorm-related programs in a virtual machine? We don't know, but we hope so.
