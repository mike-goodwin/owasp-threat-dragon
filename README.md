<p align="center">
  <img src="http://mike-goodwin.github.io/owasp-threat-dragon/content/images/threatdragon_logo_image.svg" width="200" alt="Threat Dragon Logo"/>
</p>

[![Build Status](https://travis-ci.org/mike-goodwin/owasp-threat-dragon.svg?branch=master)](https://travis-ci.org/mike-goodwin/owasp-threat-dragon) [![codecov.io](http://codecov.io/github/mike-goodwin/owasp-threat-dragon/coverage.svg?branch=master)](http://codecov.io/github/mike-goodwin/owasp-threat-dragon?branch=master) [![Code Climate](https://codeclimate.com/github/mike-goodwin/owasp-threat-dragon/badges/gpa.svg)](https://codeclimate.com/github/mike-goodwin/owasp-threat-dragon) [![SecurityHeaders.io](https://securityheadersiobadges.azurewebsites.net/create/badge?domain=https://threatdragon.azurewebsites.net/)](https://securityheaders.io/?q=https://threatdragon.azurewebsites.net/&hide=on) [![GitHub license](https://img.shields.io/github/license/mike-goodwin/owasp-threat-dragon.svg)](LICENSE.txt)

# [OWASP](https://www.owasp.org) Threat Dragon #

Threat Dragon is an online threat modelling web application including system diagramming and a rule engine to auto-generate threats/mitigations. It is an [OWASP Incubator Project](https://www.owasp.org/index.php/OWASP_Threat_Dragon). The focus will be on great UX, a powerful rule engine and integration with other development lifecycle tools.

We are currently maintaining [a working protoype](http://threatdragon.azurewebsites.net/#/) in sych with the master code branch.

**Project leader:** Mike Goodwin (mike.goodwin@owasp.org)

---

## Prerequisites

We have always attempted to make the installation of Threat Dragon as simple as possible by making all of its functionality and code available inside a single install package.  However, starting with this release, we have decided to include features within the code that interact with some third party services, which will add some additional steps to the installation process.  We thought long and hard about making these design decisions, but as the user experience is greatly improved by utilising this functionality, we felt it was something that we needed to add.  Therefore, before attempting to install Threat Dragon you will need the following;

- **A GitHub Account** - We use your GitHub account to store threat models so an account needs to be created if not already obtained.  The Threat Dragon application should be registered as a [GitHub OAuth Application](https://github.com/settings/applications/new) within GitHub.
- **An Azure Account** with **Azure Table Storage** - We use this to store session data and is necessary even if you don't intend to install the software on Azure.

## Installing Threat Dragon Code

Threat Dragon itself is a Single Page Application (SPA) using Angular on the client side and node.js on the server.  In order to build and run it locally you will need to follow these steps:

Firstly, you should install Git and node.js on the server.  You will need to locate the directory in which you wish the application to be installed and then issue the following commands;

`git init`  
`git clone https://github.com/mike-goodwin/owasp-threat-dragon.git`

These two commands will install the code in two subfolders, one for the main application (`td`) and one for the unit tests (`td.tests`).  The next step is to download and install all of the relevant node packages using;

`npm install`

(In some cases, if error messages regarding permissions are noted after you have run the above command, you will need to add `sudo` to this command.)

At this point, the main code of Threat Dragon is installed, but it will not run correctly until the following integrations are performed.

## GitHub and Azure Table Storage Integration

Once the core code is installed, the following steps should be performed to enable the final part of configuration with Azure and Table Storage.  You will need the following IDs and Secrets;

- **GitHub:** Client ID, Client Secret and Session Signing Key
- **Azure Table Storage:** Account Name and Storage Access Key

Start by setting the GitHub environment variables on the server by using the following in OSX or Linux;   

`export GITHUB_CLIENT_ID=ABCABCABCABC`  
`export GITHUB_CLIENT_SECRET=DEFDEFDEFDEF`  
`export SESSION_SIGNING_KEY=GHIGHIGHIGHI` 

Or the following in Windows;

`set GITHUB_CLIENT_ID=ABCABCABCABC`  
`set GITHUB_CLIENT_SECRET=DEFDEFDEFDEF`  
`set SESSION_SIGNING_KEY=GHIGHIGHIGHI`  
 
Once a user is signed into the application, their session information contains an OAuth access token with 
write access to their GitHub repos. For security, this is encrypted before storage in the session. The 
session encryption supports multiple keys so that they can be expired without any interruption to the 
running application. The primary key is always used for encryption. Retired keys can be kept available for 
decrypting existing sessions. Once all sessions are using the new primary key (typically this will be around > 60 minutes maximum), the old one can be safely removed. The keys are stored as a JSON string in  the 
`SESSION_ENCRYPTION_KEYS` environment variable. For example:
 
`[{\"isPrimary\": true, \"id\": 0, \"value\": \"abcdef\"}, {\"isPrimary\": false, \"id\": 1, \"value\": \"ghijkl\"}]`

As described earlier in this document, along with GitHub, Threat Dragon also uses Azure Table Storage for the session store via [connect-azuretables](https://www.npmjs.com/package/connect-azuretables). To make this work, the following Azure environment variables need to be configured in addition to the GitHub ones.  Perform the following on the server (OSX/Linux);
 
`export AZURE_STORAGE_ACCOUNT=JKLJKLJKLJKL`  
`export AZURE_STORAGE_ACCESS_KEY=MNOMNOMNOMNO` 

Or if on Windows;

`set AZURE_STORAGE_ACCOUNT=JKLJKLJKLJKL`  
`set AZURE_STORAGE_ACCESS_KEY=MNOMNOMNOMNO`  

See the [connect-azuretables](https://www.npmjs.com/package/connect-azuretables) documentation for more options.

**For Development and Proof of Concept Scenarios Only:** *If you don't want to use Azure Table Storage you 
can set the `SESSION_STORAGE`environment variale to `local`. Threat Dragon will then use the express-session in-memory session store. As [mentioned in the express-session docs](https://github.com/expressjs/session) 
this is for development only - it is not suitable for production.*

If you want to use an [alternative session store](https://github.com/expressjs/session#compatible-session-stores) in production, install it and edit the [session.config.js](https://github.com/mike-goodwin/owasp-threat-dragon/blob/master/td/config/session.config.js) file.

Finally, by default, Threat Dragon will set the `secure` flag on cookies. To override this for development purposes, set the `NODE_ENV` environment variable to `development`. 

## Running the Application

Once your environment variables are configured, start the node web server by running the following:

`npm start`

If you then browse to `http://localhost:3000` you should see the running application.

---

## Debug builds

Threat Dragon currently uses [Grunt](http://gruntjs.com/) for its build workflow, if you want to change the build, run the following on the server;

`npm install -g grunt-cli`

The default build minifies the Javascript and CSS. It does build code maps, but if you want to run with
unminified files run the following on the server;

`grunt debug`  
`npm start`

## Running the unit tests

The unit tests are written using Jasmine and Karma. Coverage is by Istanbul. A few different npm scripts are available:

* `test-client-phantomjs`, `test-client-firefox`, `test-client-chrome`, `test-client-ie`: runs client side tests using the specified browser
* `test-server`: runs the server side tests
* `test`: runs jshint, client side tests on Firefox and PhantomJS and server side tests (this is what runs on Travis CI)
* `test-local`: runs jshint, client side tests on all browsers and then the server side tests (useful as a pre-push git hook)
* `citest`: continuously runs client side tests in PhantomJS with `--single-run false` (useful while coding)

**Note:** If you are on Windows and are having problems installing Karma, the simplest way to resolve this seems to be to install Python v2.7.x (not v3+) and then install Visual Studio Express as per the SO answer suggested in [this link](http://codedmi.com/questions/298619/npm-install-g-karma-error-msb4019-the-imported-project-c-microsoft-cpp-defau). This sounds mad, but the alternative is a world of pain installing various patches and components one by one. At least it's free :o/

###Unit test coverage progress

We aim to maintain unit test coverage at > 90%

![codecov.io](https://codecov.io/github/mike-goodwin/owasp-threat-dragon/branch.svg?branch=master)

Also, the code is currently lint free :)

## Freshness

npm

[![Dependency Status](https://www.versioneye.com/user/projects/56185934a193340f2f000262/badge.svg?style=flat)](https://www.versioneye.com/user/projects/56185934a193340f2f000262) 

bower

[![Dependency Status](https://www.versioneye.com/user/projects/56185933a193340f2800026b/badge.svg?style=flat)](https://www.versioneye.com/user/projects/56185933a193340f2800026b)
