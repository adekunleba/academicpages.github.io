Delving deep into the realm of Data engineering, most especially using Scala as the programming language of choice, there seems to be a very basic thing for new entrants into the field, this applies both to the company trying to set up a data engineering platform and the engineer/consultant trying to propose a way to do this effectively. The question of how do I build my data platform right.

It is important to note that there are tools out there that help to consolidate data quickly for companies who don't mind to outsource majority of their data engineering pipeline, however I have come to notice that a lot of bigger companies and deeply technology focused companies tends to build something specific to their use cases. 

Here are the examples I have seen:
Tesla built its own data engineering platform, there is a beautiful talk and article around this from Colin breks. https://blog.colinbreck.com/the-state-of-the-art-for-iot/
LinkedIn, pluralsight,grammarly also have their own data engineering platform.
This brings the case of is it worth it building your own engine. 

I will say this really depends *but there is a joy of flexibility and long term management* that is in building personalized data engineering platform if your are pretty medium size software company.

There is the ability to integrate perfectly into your own applications as well as the ability to control the direction of flow of your overall technology.

Need I also mention though that there is a cost and time overhead to this, but the joy is when you built something incredible that you can use accross board and even build bigger application on. There is also a chance that the engineering experience will let you know how you stand as a company in the technology space.

Now for a beginner trying to enter into the landscape of Data engineering. The buzz is usually to know SQL and all, but beyond that, you need to be able to monitor and **control your data ingestion layer as well as the transformation layer**. And here is where the flexibility that we talked about comes to play. If you have a very huge user base, with several micro services working to help your user e.g the likes of linkedin or tesla or netflix then usually, there is no ingestion vendor that might likely work for your overral architecture requirements. *PS: This should however not be taken at face value.* 

Nevertheless, when starting out in the data engineering lifestyle of a company, one can leverage available tools or vendors to start but should definetly put in place a plan to build an organization's own scalable engine as the company's requirement continues to grow.

Architecting the solution for scalable data engineering platform is another challenge. It is easy to write the software to adapt entirely to your use case and less agnostic and adaptable to differing cases. Most common issues are:
- I may need to know before-hand the schema(s) of my incoming data so that I can model my application as such.
- If the incoming data schema changes, or there is a new field to be captured, does that mean we need to redeploy our data platform.

I have seen several solutions that tends to solve this problem in a way, Linkedin released Gobblin, such an impresive project but it only just helps scale your data ingestiong processes, you still need to manually write and deploy the data source connection and inner processing of the data.
Hydra is a pluralsight tool that also tends to manage schema reusability and update while sending data to various sinks. This approach has the overhead that the client needs to manage pushing the data through an API.

The challenge still remains and I believe there are still some knowledge development on how to effectively build a solution that once you submit a job you can easily schedule it and everybody is happy.


