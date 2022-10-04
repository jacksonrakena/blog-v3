---
layout: post
title: "welcome to the world, Project Spork"
tags: languages, rust, mistakes
image: /assets/media/img/2022-10-01-project-spork-announcement/congress.svg
---

Today I'm happy to announce my latest ~~obsession~~ piece of work, the beautifully-named "Project Spork".  
  
<div class="d-flex flex-column align-items-center p-4">
    <img src="/assets/media/img/2022-10-01-project-spork-announcement/congress.svg" alt="Project Spork logo" width="200" height="200" />
    <small class="gray pt-2">Project Spork's (provisional) logo</small>
</div>


It's a full-stack website and data analysis project that I've been spending brain cycles on recently, and I'm excited to share it and its lofty goals with the world.

I intend it to be very similar to [They Work For You](https://theyworkforyou.com), a similar site described as a "parliamentary monitoring website" that tracks happenings is Westminster, as well as regional authorities in the United Kingdom.

<div class="d-flex flex-column align-items-center p-4">
    <img src="/assets/media/img/2022-10-01-project-spork-announcement/twfy-voting-record.png" alt="Voting record in They Work For You" width="600"  />
    <small class="gray pt-2 text-center" style="max-width:400px">A screenshot from They Work For You, showing the voting record of a member of the British Parliament</small>
</div>

If you don't live in New Zealand (or you're a normal human that's not interested in politics) it probably won't be of use for you. My intended demographics are:

- Political wonks. (Like me.)
- News media publishing articles on a specific politician or party.
- Citizens interested in how their local representative has voted on the issues that matter to them.
- Data scientists pretending they're doing valuable work to society.

# How parliamentary voting works in New Zealand
In the Parliament of New Zealand, there is just one voting chamber: the House of Representatives, made up of ~120 members. 71 of those members are elected from an electorate, which is a physical boundary representing roughly 50,000 people. The remaining 49 members are elected from their party lists, which is chosen by the voters through their "party vote".

In Parliament, members often don't choose whether they vote 'yes' or 'no' on a resolution or bill. Often they will vote with their party, and their party will submit all of their members' votes in a block.

However, in rare circumstances, the party whips may allow a "conscience vote", which allows members to vote independently, and often occurs when a large number of members are expected to "break ranks" with their party and vote in a different way. These "conscience votes" are key to holding Parliament members to account, as they are the only way to see how individual members have voted on a particular issue.

Famous conscience votes include:
- Reforming laws relating to gay men in 1986
- Permitting same-sex couples to marry in 2013
- Liberalising abortion access in 2020

I consider it important that New Zealand voters are able to easily see how their elected representatives voted, especially on these conscience votes.

# Spork's current architecture plans
Spork is an ambitious project. In order to make it work, several moving components have to work in harmony, and there's lots of work to do.
  
Spork is generally split into three sections:

### 1. The scrapers, and associated tools
The scraping tools (`scrape_tools`) are written in Rust and use a combination of public APIs and HTML scraping to scrape data from the New Zealand Data Service, the Parliament website, and other Government websites. It is designed to be run on a regular basis, and will attempt to match votes, debates, bills, and politicians against existing data.
  
The scrape tools also include tools to maintain the database, export and import data, and perform various other tasks.

### 2. The server
The server (`server`) is a Rust web server that serves the website and API. It is written using the [Rocket](https://rocket.rs) web framework, and uses [Diesel](https://diesel.rs) to interact with the database.

The server component also includes the schema for a Postgres-based database to store scraped data. The Postgres database is designed to be easily exportable for offline and statistical analysis. All scraped data will be open-source in reasonable formats for free consumption.

### 3. The public client
The public client (`client`) is a React web app that is served by the server. It is written using [React](https://reactjs.org) and [TypeScript](https://www.typescriptlang.org), and uses [Chakra UI](https://chakra-ui.com/) for styling.

The details of the client have not been finalised, as the bulk of work at this stage is on the server and scraping tools. However, the client will be a single-page app that allows users to search for politicians, bills, and votes, and view the results.