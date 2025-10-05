# TyR projects

Collection of TyR projects and their actual state (might get outdated).

Not limited to the TyR projects themselves, also includes a list of my own persona/other projects just for inventorization.

## Latest cluster projects

These projects are migrated to the latest deployment / swarm cluster, including latest README with badges & triple-mirroring to github/gitlab:

- Habits - habit tracker
- AirCaptain - a number of apps for flight simulation
- Nitrotype Tracker - typing tracker for NitroType teams

## Production - regular docker

These apps are still on regular (legacy) docker (non-swarm), but we need them in Production, so they are moved to the new cluster too:

- Kartman - karting manager
- FoulBot - cursing Telegram/Discord bots

## The rest apps

These apps are still not migrated to the new cluster, and this list will progressively gets updated (moved to the latest cluster projects section) with time:

- OverLab - progressive overload body building tracker
- FoulTalk - an (unsuccessful) attempt to refactor/reboot FoulBot project, will be attempted again in future. I need major refactoring there / rewrite
- `tyr-authen` - authentication solution (on Duende IdentityServer) for machine-to-machine client credentials authentication for TyR apps
- `typingrealm` - main project/repo for the typing tool (currently written in html / without canvas, and just a training tool, with beginning of a game)
  - This application has numerous other repositories, at different stages of attemts of writing it, the list is below
  - `tat-front` - javascript experimentation with frontend for typingrealm project
  - `typingrealm-ng`
  - `typingrelam-prototype`
  - `typerealm-node`
  - `typerealm` - one of the first iterations of this game with TCP/protobuf networking, written during New Year Eve's night
  - `typingrealm-gems` - some gems made during typingrealm development
  - `typingrealm-old`
  - `typingrealm-web`
  - `typerealm-web-prototype`
  - `btp` - Big Typernatural Project - very basic images-driven Supernatural-inspired typingrealm game in Python no pygame3

## Other & helper repositories

These repositories live under `ewancoder` for now, but might be moved to `ewancoder-tyr` in future & add mirroring:

- `tyr-bootstrapper` - bootstrapper script to create new TyR projects
- `tyr-ops` - collection of the documentation of all my Linux/Server/Tyr knowledge
- `dotfiles` - my current linux setup dotfiles
  - `dotfiles-legacy` - my old 10+ years old configuration, before Windows 10
  - `etc` - 10+ years old /etc configuration for Arch
  - `rpi` - Raspberry PI dotfiles, 10+ years old / outdated
- `real` - latest Arch linux install script, currently heavily tailored for my needs, but still can be used as a universal tool
  - `eal` - old (legacy) outdated Arch Linux install script (EAL)
- tyr-media - media server (`*arr` stack) - deployed to local PC for easy media management
- doneman - a container that monitors, restarts and attaches other containers to specific network, we needed it for non-swarm deployments, written in Go
- healthcheck - a healthcheck binary container for simplifying healthchecks in chiseled image, written in Go
- ewancoder - project provides README file for main GitHub page, part of the CV
- ewancoder.github.io - my personal website / cv, needs an upgrade
  - website-old.2 - my old website, needed a rewrite because Dropbox changed their policy on public files. It was reading the files directly from Dropbox and did not contain any content itself
  - website-old.1 - even older version of my website, 2014
- concurrency - concurrency examples for my concurrency course
  - concurrency-legacy - an older version of the examples for a concurrency lecture
- anki-card-father - cringey name, but it's an AI anki card creator for faster creation, simple console app
- `new-eden-moons` - application to provide a list of moons scheduled for a particular day with a neat UI. Repo is private because it had private keys hardcoded in the app
- `eve-profits` - an attempt to create a trading app, died very quickly
- `wildlander` - my roleplay story (just the very beginning) of my first Wildlander playthrough
- `iracing` - my iRacing settings lol :) I might need to do this approach for other games as well, it sounds too good
- `streaks` - GTD app to manage habits; now that we have **Habits** app - this is obsolete, it was a console app
  - Although the idea was to store all `events` to analyze them later, we might need to alter the **Habits** app with the same idea for `streaks` to become completely obsolete
  `streaks-old` - oldest attempt to create it
- `async-race-sample` - example application where I demonstrated some concurrency concepts to friends
- `splash-and-dash` - simple visualization web-app for endurance racing calculation
- `learn-and-read` - the idea was to parse a book for most frequent words, mark everything I know, and learn everything I don't know starting from the most frequent ones
- DDD stuff
  - `base-classes` - some DDD-inspired base classes like ValueObject and Entity
  - `ddd-base` - more base classes for DDD with proper equality comparison
  - `lifestyle-planning` - planning application created following DDD, BDD, xUnit, Autofac, OWIN
  - `ewancoder-ddd` - large library supporting DDD, CQRS and EventSourcing
  - `ewancoder-solid` - some solid principles implemented in C#
- `larep` - my LaTex labs and diploma graduation work
  - `latex-sphinx` - my own debian image with installed latex & python-sphinx docs, not sure whether we need it
  - `tof` - Time of Flight (un)proof-of-concept app for a graduation paper
- `expert-system` - very basic python image/sounds/text - driven interactive presentation for "expert system" demo
- `tolerance` - university-made Delphi program to visually compare multiple engineering tolerances
- `vim-clutch` - a Python attempt to make a driver for a single Pedal to work as a vim button
  - this is a rewrite of a fork I also have: `software-vim-clutch` - a clutch pedal for VIM written in Python
- Angular libs (outdated) for Angular 2/4 - will use them later for creating web TyR framework
  - `angular-http`
  - `angular-notify`
  - `angular-reactive`
  - `angular-types`
  - `angular-materialize`
  - `angular-forms`
  - `angular-localization`
  - `angular-dialog`
  - `angular-animation`
  - `angular-auth`
  - `angular-logger`
- `git_quiz` - an interesting take on learning git - practical tasks; might want to re-invision this myself

## Forks

These are the forks of other people's repositories I made to not lose them if the original people decide they want to delete them.

- `pia-wg-config` - Go-written tool for generating a PIA config for WireGuard, needed for our `tyr-media` media server / gluetun vpn to work in wireguard mode
- `slog-seq` - logging solution for Serilog to work with Go
- `archive-mediatr` - MediatR fork (checkout the version that still had Apache license)
  - latest version is moved to `luckypennysoftware` org
- `archive-automapper` - Automapper fork (checkout the version that still had Apache license)
  - latest version is moved to `luckypennysoftware` org
- `keyboard-navigation-hints` - similar to SurfingKeys/cVim for browser, only for VSCode: looks super dope
- `dbmate` my go-to db migration tool
- `core-js-101` - interesting approach to learning JS - tasks written to fail/pass unit tests
- `subnautica-death-run` - hadcore mod for subnautica
- `CSharpExtensions` - some analyzers/constraint like required/initonly, by cezarypiatek
- `angular2-google-maps` - working Angular 2 google maps component with pins
- `wmii` - wmii window manager, exported from a different hosting
  - `wmii.libixp` - updated wmii / update, exported from a different hosting
  - `wmii-1` - fork of wmii (oldest) from an existing github repo
- `oh-my-zsh` - zsh library for customization

## Corporate repos

- `exadel-report-hub` - an ephemeral project for onboarding new people
- `modem-manager` - documentation (using ReadTheDocs) for modem-manager application
