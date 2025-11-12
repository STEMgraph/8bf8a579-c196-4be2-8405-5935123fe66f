<!---
{
  "id": "8bf8a579-c196-4be2-8405-5935123fe66f",
  "teaches": "Using Neo4j in a Docker Container",
  "depends_on": ["d1bee1c7-d88a-4f00-a44e-3e402f6ee826"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-27",
  "keywords": ["Neo4j", "Docker", "Container", "Persistent Storage", "Graph Database", "Cypher"]
}
--->

# Using Neo4j in a Docker Container

> In this exercise you will learn how to deploy and interact with a Neo4j graph database using Docker containers. Furthermore, we will explore how data persistence affects containerized databases and how to prevent data loss using volume mounts.

### Introduction

Neo4j is a highly scalable, native graph database designed to leverage data relationships as first-class entities. It uses a graph model for data storage and retrieval, which allows for flexible and efficient queries across highly connected data using the Cypher query language. Neo4j is commonly used for applications involving social networks, recommendation engines, fraud detection, and more.

Running Neo4j within Docker offers a portable and repeatable environment for development and testing. Like other containers, these instances are ephemeral by default. Without proper volume configuration, all database state is lost when a container is stopped and removed. This exercise focuses on running a Neo4j container first without, then with persistent storage, demonstrating the significance of data durability in containerized environments.

You'll start by running a Neo4j container, accessing its command-line Cypher shell, adding graph data, and exploring it using Cypher. You'll then see what happens to that data when the container is removed. Finally, you'll configure Docker volumes to persist Neo4j’s data and verify the setup by stopping and restarting the container.

### Further Readings and Other Sources

* [Neo4j Docker Documentation](https://neo4j.com/developer/docker-run-neo4j/)
* [Neo4j Cypher Query Language Docs](https://neo4j.com/docs/cypher-refcard/current/)
* [Docker Volumes Documentation](https://docs.docker.com/storage/volumes/)
* [Intro to Graph Databases – YouTube](https://www.youtube.com/watch?v=4jtmOZaIvS0)

### Tasks

1. **Pull the Neo4j Image:**

   ```bash
   docker pull neo4j:5
   docker images | grep neo4j
   ```

2. **Run a Neo4j Container Without Persistent Storage:**

   ```bash
   docker run --name neo4j-temp -p7474:7474 -p7687:7687 \
     -e NEO4J_AUTH=neo4j/test123 -d neo4j:5
   docker ps -a
   ```

3. **Access Neo4j via Cypher Shell and Add Data:**

   ```bash
   docker exec -it neo4j-temp bin/cypher-shell -u neo4j -p test123
   ```

   Then run:

   ```cypher
   CREATE (a:Person {name: 'Alice'});
   CREATE (b:Person {name: 'Bob'});
   CREATE (a)-[:KNOWS]->(b);
   MATCH (n) RETURN n;
   ```

4. **Explore Data Using Cypher Queries:**

   ```cypher
   MATCH (p:Person) WHERE p.name STARTS WITH 'A' RETURN p;
   ```

   Use `:exit` to leave the shell.

5. **Stop and Remove the Container:**

   ```bash
   docker stop neo4j-temp
   docker rm neo4j-temp
   ```

6. **Run a New Container With the Same Setup:**

   ```bash
   docker run --name neo4j-temp -p7474:7474 -p7687:7687 \
     -e NEO4J_AUTH=neo4j/test123 -d neo4j:5
   docker exec -it neo4j-temp bin/cypher-shell -u neo4j -p test123
   ```

   Run `MATCH (n) RETURN n;` and confirm the graph is gone.

7. **Create Docker Volumes for Persistence:**

   ```bash
   docker volume create neo4j-data
   docker volume create neo4j-logs
   docker run --name neo4j-persistent -p7474:7474 -p7687:7687 \
     -v neo4j-data:/data -v neo4j-logs:/logs \
     -e NEO4J_AUTH=neo4j/test123 -d neo4j:5
   ```

8. **Verify Data Persistence:**
   Recreate the graph data using `cypher-shell`, then stop and restart:

   ```bash
   docker stop neo4j-persistent
   docker rm neo4j-persistent

   docker run --name neo4j-persistent -p7474:7474 -p7687:7687 \
     -v neo4j-data:/data -v neo4j-logs:/logs \
     -e NEO4J_AUTH=neo4j/test123 -d neo4j:5
   docker exec -it neo4j-persistent bin/cypher-shell -u neo4j -p test123
   MATCH (n) RETURN n;
   ```

   Confirm the data still exists.

### Questions

1. What happens to the Neo4j data when the container is deleted without persistent storage?
2. Why is using a volume critical for database containers?
3. How do you list all Docker volumes on your system?
4. Can you explain the commands `-v neo4j-data:/data` and `-v neo4j-logs:/logs`?
5. What alternative methods besides named volumes could you use to persist data in Docker?

### Advice

Graph databases like Neo4j are extremely powerful for modeling connected data. However, like all stateful services, they require careful consideration when deployed in containers. Without persistent storage, all your nodes, relationships, and configurations vanish with the container. This exercise helps you appreciate the importance of Docker volumes in real-world database management. Explore advanced Cypher queries and consider integrating Neo4j with other services to simulate production-like workflows. Try this alongside other graph-processing exercises in the advanced database module \[link-to-exercise-advanced-graph-db].
