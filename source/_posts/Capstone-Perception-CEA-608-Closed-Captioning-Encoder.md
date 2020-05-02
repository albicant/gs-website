---
title: 'Capstone: Perception - CEA-608 Closed Captioning Encoder'
date: 2019-12-12 21:49:41
tags:
---
### Project Overview
During Summer and Fall 2019 at [Portland State University](https://www.pdx.edu/) I participated in the [CS Capstone](https://www.pdx.edu/computer-science/capstone) project. As part of an [8 member](https://github.com/capstone-team-a/Perception#contributors) [Agile](https://en.wikipedia.org/wiki/Agile_software_development) team, we desined a web application called ["Perception"](https://github.com/capstone-team-a/Perception) to encode closed captioning data into the [CEA-608](https://en.wikipedia.org/wiki/EIA-608) byte pairs format. This application was developed for the internal needs of [Amazon Web Services (AWS) Elemental Live](https://aws.amazon.com/elemental-live/), with consistent deliveries that met the demands of [AWS Elemental](https://www.elemental.com/) team.

### My Role: R&D Engineer
My role on this team was twofold: I was the main researcher of CEA-608 and a back end engineer. Before we had a working version, I was transforming closed captions to byte pairs by writing them out by hand. I also implemented the core functionality for creating byte pairs.

In addition, I participated in creating the initial UI design, presentations to the sponsor, twice a week [scrum](https://en.wikipedia.org/wiki/Scrum_(software_development)) meetings and testing.

### Creating CEA-608 Byte Pairs
![](/images/perception1.png)
In this screenshot, you can see the work I was doing by hand in order to understand how to work with CEA-608. The resulting byte pairs represented in the hexidecimal format (as seen above in the column "Hex") were put into AWS Elemental Live software to test that the desired caption ("I'm your Huckleberry") will be displayed with the desired style. 

I also made the [CEA-608 Reference Sheet](https://docs.google.com/document/d/1GigKq2DuLFBuGjMSP3BT_yEVJRNedL-094kt7KUj-Ds/edit?usp=sharing) for my team to develop the back end functionality.

### Project Links
You can find the project code [here](https://github.com/capstone-team-a/Perception) on github.