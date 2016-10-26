* Actors who played in some movie

MATCH (m:Movie {title: 'Forrest Gump'})<-[:ACTS_IN]-(a:Actor)

RETURN a.name, a.birthplace


* Find the most prolific actors

MATCH (a:Actor)-[:ACTS_IN]->(m:Movie)

RETURN a, count(*)

ORDER BY count(*) DESC LIMIT 10;

* Find actors who have been in less than 3 movies

MATCH (a:Actor)-[:ACTS_IN]->(m:Movie)

WITH a, count(m) AS movie_count

WHERE movie_count < 3

RETURN a, movie_count

ORDER BY movie_count DESC LIMIT 5;

* Find the actors with 20+ movies, and the movies in which they acted

MATCH (a:Actor)-[:ACTS_IN]->(m:Movie)

WITH a, collect(m.title) AS movies

WHERE length(movies) >= 20

RETURN a, movies

ORDER BY length(movies) DESC LIMIT 10;

* Find prolific actors (10+) who have directed at least two films, count films acted in and list films directed

MATCH (a:Actor)-[:ACTS_IN]->(m:Movie)

WITH a, count(m) AS acted

WHERE acted >= 10

WITH a, acted

MATCH (a:Director)-[:DIRECTED]->(m:Movie)

WITH a, acted, collect(m.title) AS directed

WHERE length(directed) >= 2

RETURN a.name, acted, directed

ORDER BY length(directed) DESC, acted DESC;

* Rewritten to filter both :Actor and :Director labels up front

MATCH (a:Actor:Director)-[:ACTS_IN]->(m:Movie)

WITH a, count(1) AS acted

WHERE acted >= 10

WITH a, acted

MATCH (a:Actor:Director)-[:DIRECTED]->(m:Movie)

WITH a, acted, collect(m.title) AS directed

WHERE length(directed) >= 2

RETURN a.name, acted, directed

ORDER BY length(directed) DESC, acted DESC;

* Using the lowest cardinality label, :Director

MATCH (a:Director)-[:ACTS_IN]->(m)

WITH a, count(m) AS acted

WHERE acted >= 10

WITH a, acted

MATCH (a)-[:DIRECTED]->(m)

WITH a, acted, collect(m.title) AS directed

WHERE length(directed) >= 2

RETURN a.name, acted, directed

ORDER BY length(directed) DESC, acted DESC;

* User-Ratings

MATCH (u:User {login: 'Michael'})-[r:RATED]->(m:Movie)

WHERE r.stars > 3

RETURN m.title, r.stars, r.comment

* Recommendations including counts, grouping and sorting

MATCH (u:User {login: 'Michael'})-[:FRIEND]-()-[r:RATED]->(m:Movie)

RETURN m.title, avg(r.stars), count(*)

ORDER BY AVG(r.stars) DESC, count(*) DESC

* Ratings by like-minded people

MATCH (u:User)-[r:RATED]->(m:Movie)<-[r2:RATED]-(likeminded),

      (u)-[:FRIEND]-(friend)

WHERE r.stars > 3 AND r2.stars >= 3

RETURN likeminded, count(*)

ORDER BY count(*) desc LIMIT 10;

* Mutual Friend recommendations

MATCH (u:User {login: 'Michael'})-[:FRIEND]-(f:Person)-[r:RATED]->(m:Movie)

WHERE r.stars > 3

RETURN f.name, m.title, r.stars, r.comment
