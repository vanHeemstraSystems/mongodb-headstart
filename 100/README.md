# 100 - Introduction

I tend to have about a dozen or so little tech projects loaded onto my machine at any given time — little web app ideas that I work on and stop, an occasional hackathon entry, something I’ve built working through a tutorial, a few experiments. I’ve gotten to really enjoy working with MongoDB over the past year, and I’ll usually add that in as a persistence layer when appropriate. One thing I’ve realized working off of a local MongoDB development server is all of those databases can accumulate, making it a little scary to experiment with different server configurations and deployment setups for one app. Indeed, one of the challenges during local development is constantly changing and reconfiguring the database as the project evolves, and safely dropping, rebuilding, and reseeding databases.

One way I’ve found to add some flexibility to the database layer when developing locally is using Docker to run a database server that’s dedicated to the project. By leveraging volumes with Docker, we can quickly configure, change, and migrate a database, making it easier to keep database configurations consistent in higher environments. Likewise, this setup can also simplify creating and tearing down pre-populated databases for automated tests.

In this article, I’ll show you my Docker setup for container configurations and scripts to run a project’s MongoDB server in a Docker container. We’ll add some configuration files for a local database, then go on to seed some data, as well as simple commands to tear down and rebuild the database.