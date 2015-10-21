# Collaboration Tools

We all have our own styles when it comes to communicating, managing our time, prioritizing things and using tools at hand. That's why it is important that we set a minimum threshold of how we as a group should use collaboration tools so that they remain effective, clear and fullfil their purpose.


##Slack
This is by far the tool that we use most at Lattice. Slack its feature packed so lets make sure we use them in unison.

###Public Channels
Most conversations should happen in these open channels. We pride ourselves from being open and transparent, and having relevant conversations in a fashion that other team members can leverage the information is key to that. That being said, not everything that goes on in a channel needs to be actioned by everyone if its not relevant to them, therefore make sure to address the person(s) you need by calling out their user handle _@xxx_ and they'll be notified.

###Notifications
Since conversations are mostly open, it can be annoying to keep Slack's default notification system in which your phone / laptop ping you every single time a message is posted. Therefore you can consider going into the settings section and setting it to a mode that is suitable for you, (i.e. to only notify you when someone mentions you, or sends you a 1-1)

###Private Groups
Certain things might be better of shared privately, for example, you might want to have a channel to discuss relevant trending open source projects. While super interesting maybe is best to let everyone know that there is a channel for that, and to request an invite. This helps keep the public channel sidebar tidy.

###1-1 Messages
Do it only if you must - we want to be part of the fun too!

###File Sharing
You can actually upload files directly into slack, and that's awesome for most things, but since we use Google Drive primarely for document storage it can get messy real quick.

If it's something temporary, or for context/reference feel free to upload it within Slack. However, is it's an ongoing document upload it fist to the GDrive team folder and share the link on Slack. It becomes easier to keep track of versions that way.

###Video Conferencing
If you feel like jumping on a call with others, we have a Goolge Hangouts integration on Slack. Just type _/Hangouts_, click the link, and the hangout session will be created and posted in the channel/group/conversation 

##Google Apps
There are 3 main google apps we use at Lattice

###Drive
Drive is your file storage service. Make sure to download the desktop and mobile application so its easier to use it.

We have a public team share folder, as a transparent team, most things should be in there. Make sure to keep it tidy and well structured.
 
You should **never** change access/sharing permissions of these folders/files to people outside the organization. If you must, ask one of the founders.

###Docs
Try and use Google Docs as much as possible. It makes collaborating on documents a hell of a lot easier. Uploading word/excel/ppt makes it both difficult for people to collaborate as well as keeping track of versions.

Google Docs has a "comment" feature, this is super useful when making suggestions on documents you are working with others. Try to use that instead of Slack for example, purely because it allows others looking at the doc to understand its history.  

###Hangouts
We use Hangouts for Video Conferencing. 


##Jira
When discussing new features, working on existing ones, or preparing for a release please ensure Jira is kept up to date. For instance;

* A version should be a target release. This is the highest level of granularity
* Every version/release contains a set of Epics (aka. high level features)
* Every Epic should be thoroughly documented with a descriptive user story, an _dev lead_ assigned to it, a priority, etc, etc.
* Each epic will come the container for a set of tasks. Each task should be a small, isolated piece of work.
* It is paramount that each task is well described, prioritized, and an estimated effort is assigned to it
* All tasks get automatically placed in the backlog.
* At the beginning of each week the team should decide which tasks will be selected for that week's sprint.
* A Sprint is a time delimited period in which the team's goal is to complete the selected subset of tasks from the backlog
* On completion, each task should be updated with the commitSHA as well as be moved to the testing / done section


##Wunderlist
Some of us use Wunderlist as a _todo_ list for each one of us and the team. Since we use Jira for engineering tasks, Wunderlist is mainly used at Lattice for operations oriented tasks.

For Example;

* Follow up with X regarding API access
* Complete Employee Handbook
* Find tickets to the X conference

One very important thing tho, is to try and keep the list prioritized so that others know what your plans are, if they can help, etc. You should always try to;
 
* Assign a due-date for todo items
* If critical, 'star' the items on your list so that it floats to the top
* Make use of checklists, notes and comments sections for items if you need to
* Assign it to others if necessary, or just move it to their folder
* If an item is on the _Anyone_ list and you fancy doing it, assign it to yourself and set priorities for it. 


##Github
There are a few things that are key while working with the codebase. It's important we all follow these guidelines
  
* Before you begin any work, make sure you have an associated Jira task created for it and it should be placed under _in progress_
* Always work against a branch. This is the safest way for us to maintain the mainline code stream stable
* Always write, update or delete any relevant unit test / integration tests that as relevant to your commit
* Each commit should be descriptive of what the issue was and how was it fixed by the commit
* Create a pull request and assign it to another relevant member of the team for code review
* All commits need to go through the Continuous Integration / Continuous deployment pipeline. 
* Make sure you make any necessary changes to the Continuous Integration system dependencies if necessary
* Once finished, ensure the Jira task associated has the relevant commit information and any other information necessary. Move it to the _done_ section
