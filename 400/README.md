# 400 - Conclusion

And with that, we’ve created a MongoDB server that communicates with a Node.js application from a container. This gives our application a dedicated MongoDB instance that’s easier to set up and tear down. We’ve also got a good foundation for getting configurations right in local development before replicating those consistently in higher environments. Lastly, we can re-configure and evolve the database server as the application changes without affecting other applications that would otherwise share the database server.
