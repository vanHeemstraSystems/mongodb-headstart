# 200 - Requirements

In order to test out the Docker scripts, you’ll need [Docker Desktop](https://www.docker.com/products/docker-desktop) running on your local development machine. If you want to follow along with the sample repo, you’ll also need [Node.js](https://nodejs.org/) 12v+ installed. Lastly, a database browser tool like [MongoDB Compass](https://www.mongodb.com/products/compass) will be helpful for testing and inspecting the databases that we build.

To demonstrate, I’ve created [a simple MEAN-stack repo](https://github.com/jsheridanwells/MeanUrls) and I’ll show how to add scripts for containerizing the database. However, you could also easily carry out these steps in any other project that uses Mongo — I’ll just be going over the Docker setup in this article. The example repo uses Angular, Express, and Typescript, but you don’t need to be familiar with any of those to follow along.

I won’t be explaining any MongoDB concepts, but that’s not likely to get in the way of understanding the steps below.

I’ll give a brief overview of some Docker terms and concepts so that the explanations are clear, but if you’re still getting started with Docker, [there are far better tutorials out there](https://docker-curriculum.com/).
