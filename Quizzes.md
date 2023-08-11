# Quizzes 

1. Let's start with creating a table that provides the following details: actor's first and last name combined as `full_name`, film title, film description and length of the movie. How many rows are there in the table?
- **SOLUTION**
    
    ```sql
    SELECT first_name, last_name,
           CONCAT(first_name, ' ', last_name) AS full_name,
    	     film.title,
    	     film.length
    FROM actor
    INNER JOIN film_actor ON film_actor.actor_id = actor.actor_id
    INNER JOIN film ON film.film_id = film_actor.film_id
    ```

2. Write a query that creates a list of actors and movies where the movie length was more than 60 minutes. How many rows are there in this query result?
- **SOLUTION**

    ```sql
    SELECT CONCAT(ac.first_name, ' ', ac.last_name) AS full_name,
           fi.title AS film_title,
	       fi.length AS film_lenth
    FROM actor ac 
    INNER JOIN film_actor fi_ac ON fi_ac.actor_id = ac.actor_id
    INNER JOIN film fi ON fi.film_id = fi_ac.film_id
    WHERE fi.length > 60
    ORDER BY fi.length;
    ```

3. Write a query that captures the actor id, full name of the actor, and counts the number of movies each actor has made. (*HINT: Think about whether you should group by actor id or the full name of the actor.*) Identify the actor who has made the maximum number movies.
- **SOLUTOIN**

    ```sql
    SELECT ac.actor_id, 
           ac.first_name || ' ' || ac.last_name AS full_name,
	       COUNT(*) AS film_count
    FROM actor ac
    INNER JOIN film_actor fi_ac ON fi_ac.actor_id = ac.actor_id
    INNER JOIN film fi ON fi.film_id = fi_ac.film_id
    GROUP BY 1, 2
    ORDER BY 3 DESC;
    ```

4. Write a query that displays a table with 4 columns: actor's full name, film title, length of movie, and a column name "filmlen_groups" that classifies movies based on their length. filmlen_groups should include 4 categories: 1 hour or less, Between 1-2 hours, Between 2-3 hours, More than 3 hours. **Match the `filmlen_groups` with the movie titles in your result dataset**.
-**SOLUTION**

    ```sql
    SELECT CONCAT(ac.first_name, ' ', ac.last_name) AS full_name,
           fi.title AS film_title,
	       fi.length AS film_length,
	   
           CASE 
                WHEN fi.length < 60 THEN '1 hour or less'
                WHEN fi.length <= 120 THEN 'Between 1-2 hours'
                WHEN fi.length <= 180 THEN 'Between 2-3 hours'
                ELSE 'More than 3 hours'
           END AS filmlen_groups
	   
    FROM actor ac
    INNER JOIN film_actor fi_ac ON fi_ac.actor_id = ac.actor_id
    INNER JOIN film fi ON fi.film_id = fi_ac.film_id
    ORDER BY 3;
    ```

5. Now, we bring in the advanced SQL query concepts! Revise the query you wrote above to create a count of movies in each of the 4 filmlen_groups: 1 hour or less, Between 1-2 hours, Between 2-3 hours, More than 3 hours. **Match the count of movies in each `filmlen_group`.**
**SOLUTION**

    ```sql
    WITH film_groups AS(
    SELECT CONCAT(ac.first_name, ' ', ac.last_name) AS full_name,
           fi.title AS film_title,
           fi.length AS film_length,
        
           CASE 
                WHEN fi.length < 60 THEN '1 hour or less'
                WHEN fi.length <= 120 THEN 'Between 1-2 hours'
                WHEN fi.length <= 180 THEN 'Between 2-3 hours'
                ELSE 'More than 3 hours'
           END AS filmlen_groups
            
    FROM actor ac
    INNER JOIN film_actor fi_ac ON fi_ac.actor_id = ac.actor_id
    INNER JOIN film fi ON fi.film_id = fi_ac.film_id
    ORDER BY 3)

    SELECT DISTINCT(filmlen_groups),
           COUNT(*) OVER (PARTITION BY filmlen_groups) film_cat_count
    FROM film_groups
    ORDER BY 2;
    ```