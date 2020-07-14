---
layout: post
title: A Paltry paradise
description: Escape the entanglement
image: /images/island.jpg
---

## I've never had a good experience developing software on Windows.

The terminal is clunky, the filesystem quirky, and that's after fighting to setup and update your environment. You often need admin access to perform everyday dev tasks. Corporate IT frequently blocks you from doing so to protect you from the overwhelming amount of malware targeted at your machine. At least this was my experience trying to get any work done in that environment.

I became so frustrated that I started looking for a better way. I was able to find portable apps for some, but not all of the tools I used. There was no single source for downloading all of them, though, and I was still on the hook for configuring each one whenever I was setting up a new environment. Best case scenario - getting up and running would waste a full day due to all this. That time adds up if your team is multiplying, you ever upgrade machines, or you want to keep the tools you're using up-to-date.

## scripted setup

The first chapter in this saga starts with my attempts to automate this process using scripts. I wrote batch files that would download and extract the latest versions of the tools I needed (if not already installed) then run my primary tool of choice with other tools available in the path. For Java development - this meant reverse-engineering how to extract the JDK files from the installer, then Maven, and finally Eclipse, after which I would launch Eclipse with all the right environment variables set to do Java stuff. All of these files would write to the current folder (presumably where my user had access) and won't touch any of the files, registry keys, or system settings I can't access. A separate script did the same thing for Node.js, npm, and VS Code - which launched at the end.

Each of my scripts started to get rather long and unwieldy, and they could not automate my configuration steps. I attempted to bolt-on more features for git support with automatic cloning for repo directories that didn't exist yet, Maven password encryption, configuring some Eclipse workspace settings, custom Node.js versions, and more. This configuration resulted in variables at the top of each script to customize per environment. It felt awkward that I had to run the script and wait for it not to find any updates to start my editors. I was also running into issues when I wanted to update one of the scripts that were already in use with custom values at the top, so the upgrade process was far from seamless. Using single-file scripts had been convenient at first, but now it was time to rethink my whole approach.

## [Paltry](https://github.com/paltry/paltry) paves a path to paradise

It was time to enter the modern era with my next project - codenamed Paltry. This time around, I would be splitting up the work for different tools into separate scripts, providing config through a [`config.json` file](https://github.com/paltry/paltry#configuration), and adding shortcuts for the various tools to launch them independently. I also offered a console with the same `PATH` and other environment variables for running CLI commands. In addition to the union of the tools I already supported - I added more - such as Atom, Chromium, OpenSSL, Ruby, and more. I made the design decision to write the core logic in PowerShell since it provides a much richer set of features than my previous batch scripts.

My new distribution mechanism involved making the project open-source and available on GitHub, then downloading the current `master` branch as a ZIP, extracting the archive to a writeable location using a built-in OS feature, and kick off the first build. Thus getting started only requires a browser, internet connection, and optionally a `config.json` for personalizing your experience. Paltry's functionality was refactored into modular plugins that package the logic needed to install, update, and configure each tool in the Paltry toolkit. The data in `config.json` determines which plugins are enabled and provides configuration data to each one as they execute. Since the code for Paltry lives in a public git repo, I added functionality in the git plugin for auto-updating your Paltry files in-place, preserving your `config.json` contents. Now the install and update process was streamlined to the point of giving me and my users superpowers over manually maintaining their dev environments.

![](/images/superhero.jpg)

## lessons in heroism

Many of my early challenges with Paltry centered around permissions issues. Overzealous system admins, greedy antivirus software, and more conspired to prevent many of the things I was attempting to do. On some locked-down machines, PowerShell would have highly restrictive default settings. This limitation was easy enough to bypass by kicking off Paltry with a batch script that used `powershell -ExecutionPolicy Bypass`. Antiviruses became notorious for holding on to file handles thanks to realtime scans, which got in the way of moving around the files each of the tool plugins downloaded and extracted. The workaround for this issue ended up being to copy the files instead and later clean up the files that weren't needed anymore.

When writing plugins for the various tools, I used proper APIs to get the latest release whenever possible. Half of my tools were available as GitHub releases, and that worked great for this. The other half, I usually had to resort to scraping the download page, looking at the links. This approach can be finicky and easily breaks if the site changes. Both suffer from finding which of the files to download for a release. Often these tools support multiple platforms, sometimes with 32 and 64-bit versions. Finding the right file to download and extract from this list introduces another potential source of unreliability. I made an effort to pick the most permissive patterns to match for these as a result.

Automating configuration for some tools was more straightforward than others. VS Code was easy since configuration lives in a `settings.json` file. Eclipse was on the opposite end of the spectrum, with lots of nested folders full of files in different formats. I ended up reverse-engineering the resulting changes to files in the workspace for each change I made. I would take a snapshot of my Eclipse workspace files before and after applying a configuration change, then diff those directories to find what changed. The plugin script was updated to apply each of these changes until I achieved all the customizations I wanted.

## leverage your legerity

The default current working directory when launching a Paltry console is the parent of the Paltry install. If you spend most of your terminal time in a different location, feel free to update `cwd` in your config. In addition the config supports `env`, which sets environment variables using `{"variable": "value"}`. There's also a `path` config option that you use to append additional tools you've manually installed outside of Paltry. The path property merges correctly with Paltry's other tools, and `PATH` should _not_ be used in the `env` config.

Paltry includes support for writing scripts, and contains a useful one out of the box: `paltry-cleanup`. Run this command from your Paltry console anytime you want to remove older tools that are no longer in use. This cleanup isn't automatic to support the cases where you fix the version of a tool to a previously-installed one. I've used this myself after the automatic update updated to a broken version of a tool. Define your scripts with an object that maps the command alias to an array of content lines in the script. Keep in mind that these will run as batch scripts, and if you want access to PowerShell, you will need to include that as part of your command.

Here are a few scripts I've written to give helpful aliases for working with Node.js projects:

```json
{
  "scripts": {
    "npmr": ["rmdir /s /q node_modules"],
    "npmi": ["npm install"],
    "npmc": ["npmr && npmi"],
    "npms": ["npm start"],
    "npmt": ["npm test"]
  }
}
```

The philosophy behind Paltry empowers Windows developers to control their environment, even with software that typically requires bribing your admins. This type of thinking is similar to life in the \*nix world, where portability is taken more seriously, and users other than `root` can get useful work done. I applied the best practices of DevOps to the local development environment and reaped the resulting benefits. Paltry even provided all the tools I used to write, preview, and publish this article.

## take a [Paltry](https://github.com/paltry/paltry) step and find your paradise
