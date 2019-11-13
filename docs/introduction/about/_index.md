---
title: "About This Training"
weight: 1
---

## Introduction

This training is designed based on the [Docksal Docs](https://docs.docksal.io) with examples and scenarios for using Docksal for local development. This is a hands on training workshop broken down into the following schedule.

* DOCKER OVERVIEW
  * Docker Basics
      * A high-level overview of Docker, how Docker works, and the main components of Docker.
  * Why Containerize?
      * What are the advantages of using containers for development vs local *AMP stacks?
  * Why Use More Than Just Docker?
      * Why should we consider using something more than just Docker to containerize?  What are the advantages to using something like Docksal to wrap up all of our Docker functionality?  In this section we'll go over how using a dev environment like Docksal makes development easier and faster compared to vanilla Docker.
* INTRO TO DOCKSAL
  * What's a Docksal?
      * Where did it come from? Was it handed to us by aliens in hopes that it would change the world one local dev environment at a time? Nope, but we will cover where Docksal came from and what differences there are between Docksal and some of the other containerized dev environments out there.
  * Docksal Stacks
      * Let's take a look at what a stack is and why we have them.
  * Docksal System and Default Services
      * We're going to explore some of the services that come with Docksal and where they're used.
  * Boilerplates and Starterkits
      * It's not just for Drupal! Let's explore a few of the available boilerplate projects that are in the Docksal Github Repo
  * What's in the Container?
      * We're going to explore some of the tools that are included with Docksal straight out of the box. Many should feel very familiar.
* GETTING STARTED
  * Installing Docksal
  * Your First "fin"
      * Let's make sure everything is installed correctly and that you're able to get Docksal running and see the the system information.
  * Starting a new project
      * We're going to use `fin` to spin up a boilerplate project and see what happens.
  * Spinning up a Drupal Site
      * We're going to spin up a basic Drupal 8 site using the Docksal Drupal 8 boilerplate and take a look at some of the things that Docksal needs to run within a Drupal codebase.
* GOING FURTHER
  * Customization
      * How to alter settings and configuration to make our system work how we want it to work.
  * Advanced Customization
      * When a stack only gets you 90% of the way there, you might need just a little more to get you the rest of the way.  We'll explore some options for making customizations and tweaking existing services to do what you need them to do.
  * Keep it Local
      * Not all settings need to make it into your repo.  In fact, it's better if some don't so that you don't accidentally push an API key into a public repo. We're going to find out how to make sure we keep private stuff on our local environment only.
  * Adding Docksal to an Existing Project
      * Let's take an existing project that we've been working on and add Docksal to it to mimic our hosting environment.
  * Addons, Addons, Addons
      * Using addons to make your life easier.  We'll explore some of the available addons, what they do, and how to install them.
  * Docksal: It's Not Just for Local Anymore
      * Let's explore some other uses for Docksal that aren't your local machine. Docksal can be used for CI builds and sandbox environments.
* Q & A AND TROUBLESHOOTING
  * Towards the end of the day we'll open up the conversation and see if there are any unanswered questions or issues that are preventing you from using Docksal in your everyday life.
