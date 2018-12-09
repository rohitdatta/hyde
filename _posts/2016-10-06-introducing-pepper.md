---
layout: post
title: Pepper - A hackathon registration system
---

This past summer, I embarked on an ambitious project for HackTX.
We wanted to design a brand new hackathon registration system that would serve as the beating heart for our hackathon.

## Rationale
Now you might be wondering why we wanted to do this.
There's numerous options out there, and building a completely new application seemed excessive.
Our tech team evaluated the options out there to get us to this point.

### Features Needed
* **Capacity**: We needed the application system to handle a *LOT* of users. HackTX has a very unique problem, being the largest hackathon based on FIFO.
As a result of this, we have recorded up to 1000 simultaneous users attempting to access our servers when launching registration.
* **Support for MyMLH**: We wanted to enable support for [MyMLH](https://my.mlh.io).
We use MyMLH to prevent hackers from needing to retype in their information from year to year.
* **Resume collection**: Every hackathon collects resumes.
But, everything is bigger in Texas, including the number of resumes we collect.
Recruiters had let us know they were having some trouble going through the vast collection of resumes we had.
So we wanted to build a way for recruiters to be able to search and filter through resumes.
* **Hub for apps**: We wanted to tie together the hacker experience.
Since 2015, we have published companion mobile apps.
We wanted our system to be able to serve up the server resources for those mobile apps, powering push notifications and the data store.
* **Volunteer registration**: We're incredibly lucky at HackTX to have so many dedicated and enthusiastic volunteers.
Managing these volunteers became a task in itself, and we wanted to automate this process, registering volunteers to use [Electron](https://github.com/hacktx/electron)

## Alternatives Considered

### Nucleus (HackTX 2015)
[Nucleus](https://github.com/hacktx/nucleus) was built by the HackTX Tech Team in 2015.
We learned a lot from Nucleus, and our original plan was to iterate upon Nucleus.
However, we quickly realized that we mistake of starting the project, before fully realizing the scope of the project.
As a result, in order to fix many of the issues, we would need to rewrite a good chunk of the code.
It removed one of the primary advantages of using the existing system.

Another concern for us was the language of choice, [Hack](https://hacklang.org/).
Now, the merits of Hack (and PHP) can be debated elsewhere, but the fact is that many hate anything to do with PHP, and we're not trying to start a language war.
No one on the current team has any experience with Hack, and shortly after looking at the code, it appeared readily apparent, that there was no interest in learning it.

### Google Forms/Qualtrics/Typeform
Another option we considered was using any of the numerous form builders out there.
We quickly decided against this for a number of reasons.
While it could easily handle whatever load we threw at it, it restricted access to the underlying data.
We couldn't as easily customize our system to meet our unique demands.
Additionally, while this would handle hacker registration, it wouldn't be able to do much else, and we would need to build our own API layer to allow separate services to communicate with this data information.

## Solution

Enter pepper (🌶). Pepper is part of the next generation of HackTX apps that we're open sourcing. Our previous atom-based naming scheme is going to be deprecated in favor of our new spices, all tied together by pepper. Being able to design this system from the ground up allowed us a lot of flexibility to make certain things work the way you want.

One of our goals with pepper is for it to not just be a HackTX project, but a community project.
We want it to be dead simple for any hackathon to launch a full-featured hackathon registration system.
So based on our wholly unscientific selection bias of the most common web framework at hackathons, we chose Flask.
We hope that any hackathon team that wants to can quickly and easily set up pepper for their own event.
Pepper is still in beta, so it's still somewhat designed to adopt HackTX branding, but we hope in the future to make it so you just need to change a few environment variables to launch your own instance of pepper.
We also made Pepper designed for deployment on Heroku. Heroku is the quickest path to deployment, and allows hackathons to easily scale the number of dynos to match their load.
However, if you want to, pepper should work well on AWS EC2, Digital Ocean droplets, or whatever is your infrastructure provider of choice.

Pepper was built to scale. Our size means that we have to know how to handle size.
I'll readily admit that running Flask isn't the *fastest* server in existence, but in our experience load testing, throwing thousands of clients against pepper run on a gunicorn server and it can handle it.
In fact, most of the limitation comes from the database connection.
Flask uses connection pooling to help mitigate this, but it should be noted that Heroku does have a limit of 500 simultaneous connections even on their highest tier of Heroku Postgres.
Our solution was to use AWS RDS, and things have been fixed.

As I mentioned above, pepper was not only built for performance, but also featureset.
We added a couple features to our system that we believe will benefit the hackathon community.
Pepper uses AWS S3 for blob storage to store and organize resumes, drastically reducing expenses for a hackathon compared to disk storage of the server.
Pepper includes a recruiter portal, and based on our talks with recruiters, they have loved it!
Some companies want to run queries to find only seniors that would be open to full-time roles, while others are seeking freshman to fill specific internship programs.
No matter the case, recruiters are now able to find the attendees they want.

Another key feature pepper brings over from pepper includes our integration with our mobile apps.
Similar to many hackathons, we create an event slack for our attendees to join. When we want to make an announcement, we do so in slack, and this triggers a workflow that sends out a push notification to anyone with our app.
This extends across the app, allowing us to update the event schedule on the go, changing event details as things happen throughout the event.

Pepper also manages our volunteer database, handling authorization for event check in.
Hackathon organizers should add the volunteer's email to the database, which they then use for signing in.
Electron, the HackTX application for attendee check in then pings the pepper API to confirm attendee details and match IDs.
Once the volunteer verifies everything, they can send back an authenticated request to pepper marking the user as checked in, allowing event organizers to gain real time insights into the check in process.

## Where we're heading

I'm incredibly excited about the future of pepper. This is just the start, and we have lots of room for improvement.
Firstly, I think we plan to move non critical features to a background queue on Celery.
Using worker processes to handle heavy non-critical tasks will enable HackTX to handle the significant volumes of traffic that we generally deal with.

We also expect a lot of work to be done on the UI, especially for admins.
I think we rightly prioritized the UI for attendees and sponsors, and put the admin interface on the backburner, but I think now that we've got the initial product out, we should make pepper easy for all users.
