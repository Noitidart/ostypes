## Table of Contents

- [How to download this into your addon](#how-to-download-this-into-your-addon)
	- [As a Submodule](#as-a-submodule)
		- [Updating the Submodule](#updating-the-submodule)
	- [I don't use git](#i-dont-use-git)
- [Platform Detection Methods](#platform-detection-methods)
	- [Off of Main Thread - Chrome Worker](#off-of-main-thread---chrome-worker)
	- [Main Thread](#main-thread)
		- [OS Global](#os-global)
		- [Services.jsm](#servicesjsm)
- [How to import the modules into your addon files](#how-to-import-the-modules-into-your-addon-files)
	- [Off of Main Thread - Chrome Worker](#off-of-main-thread---chrome-worker-1)
	- [Main Thread](#main-thread-1)
- [Usage after importing the files](#usage-after-importing-the-files)
- [Why cutils.jsm and ctypes_math.jsm?](#why-cutilsjsm-and-ctypes_mathjsm)

## How to download this into your addon
### As a Submodule
The ostypes repository is meant to be used a submodule. You must be using git. To include it as a submodule open git shell and execute the following:

    git submodule add git@github.com:Noitidart/ostypes FOLDER_NAME_HERE

This will add ostypes repo to your root addon directory, into a newly made folder named `FOLDER_NAME_HERE`. You can change that to whatever you want. You can use a path o a sub-folder such as `blah1/blah2/rawr`. This will create the directory "blah1" and "blah2" if they don't exist. Then it will create and download the contents into the "rawr" folder.

#### Updating the Submodule
The ostypes repository is very welcome to pull requests and updates. I update it myself as needed. To update the ostypes submodule that you had previously already included, `cd` into the sub-folder you downloaded ostypes submodule to, and then and then do `git pull`. This will update it to the `master`.

### I don't use git
You don't have to use this as a submodule. Simply download the repository, then put the files wherever you want.

## Platform Detection Methods
We are using platform dependent APIs so it is important to detect the platform. With both methods below, the list of of possible values is seen here - [MDN :: OS_TARGET](https://developer.mozilla.org/en-US/docs/Mozilla/Developer_guide/Build_Instructions/OS_TARGET).

### Off of Main Thread - Chrome Worker
By default, a `OS` global exists in `ChromeWorker`s, we will use this to determine the platform. We can access the operating system name with `OS.Constants.Sys.Name`.

### Main Thread
#### OS Global
We can access the system name in the same `OS` global on the main thread by importing the `OS.File` module. We won't use all the features of the module, We will only access `OS.Constants.Sys.Name` to figure out the platform.

#### Services.jsm
The `Services.jsm` module has a lot of commonly used features. This is usually always imported to addons and the preferred way to get access to the same `OS_TARGET` variable. This is preferred to `OS.File` module because we don't use 99% of what comes with that module when all we need to do is get the platform name. After importing Services.jsm we can access the platform name with `Services.appinfo.OS`.

    const {utils: Cu} = Components;
    // var { Cu } = require('chrome'); // if you are using Addon SDK
    Cu.import('resource://gre/modules/Services.jsm');

    var os_name = Services.appinfo.OS.toLowerCase();

## How to import the modules into your addon files
### Off of Main Thread - Chrome Worker
You cannot use js-ctypes from a Web Worker, it must be a Chrome Worker. This ostypes module is meant to be run from a ChromeWorker, however some API methods need to be run on the main thread. The code below shows you how to import it into the main thread, it assumes the ostypes repo was download into your addon at a sub-folder in your root directory at path `./modules/ostypes`.

_**Note for Linux/Unix Systems** `ostypes_x11.jsm` is meant for *nix sysstems. GTK and X11 should only be used on the main thread, it will causes unexplainable crashes if used from a ChromeWorker. Inside `ostypes_x11.jsm` though is XCB, this is what you want to use if you want to do platform stuff on *nix systems from a ChromeWorker._

    var GLOBAL_SCOPE = this;

    function importOstypes() {
      importScripts('chrome://myaddon/content/ostypes/cutils.jsm');
      importScripts('chrome://myaddon/content/ostypes/ctypes_math.jsm');

      var os_name = OS.Constants.Sys.Name.toLowerCase();
      switch (os_name) {
          case 'winnt':
          case 'winmo':
          case 'wince':
                  importScripts('chrome://myaddon/content/ostypes/ostypes_win.jsm');
              break;
          case 'darwin':
                  importScripts('chrome://myaddon/content/ostypes/ostypes_mac.jsm');
              break;
          default:
              // we assume it is a GTK based system. All Linux/Unix systems are GTK for Firefox. Even on Qt based *nix systems.
              importScripts('chrome://myaddon/content/ostypes/ostypes_x11.jsm');
      }
    }

### Main Thread
Sometimes it is necessary to use on the main thread, because certain platform APIs require to be run on the main thread.

We will detect the platofrm, then we will import the the respective ostypes file.

    const {utils: Cu} = Components;
    // var { Cu } = require('chrome'); // if you are using Addon SDK
    Cu.import('resource://gre/modules/Services.jsm');

    var GLOBAL_SCOPE = this;

    function importOstypes() {
      Services.scriptloader.loadSubScript('chrome://myaddon/content/ostypes/cutils.jsm', GLOBAL_SCOPE);
      Services.scriptloader.loadSubScript('chrome://myaddon/content/ostypes/ctypes_math.jsm', GLOBAL_SCOPE);

      var os_name = Services.appinfo.OS.toLowerCase();
      switch (os_name) {
          case 'winnt':
          case 'winmo':
          case 'wince':
                  Services.scriptloader.loadSubScript('chrome://myaddon/content/ostypes/ostypes_win.jsm', GLOBAL_SCOPE);
              break;
          case 'darwin':
                  Services.scriptloader.loadSubScript('chrome://myaddon/content/ostypes/ostypes_darwn.jsm', GLOBAL_SCOPE);
              break;
          default:
              // we assume it is a GTK based system. All Linux/Unix systems are GTK for Firefox. Even on Qt based *nix systems.
              Services.scriptloader.loadSubScript('chrome://myaddon/content/ostypes/ostypes_x11.jsm', GLOBAL_SCOPE);
      }
    }

## Usage after importing the files
...

## Why cutils.jsm and ctypes_math.jsm?
...
