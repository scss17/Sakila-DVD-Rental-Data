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

# Investigate Relational Database
6. We want to understand more about the movies that families are watching. The following categories are considered family movies: Animation, Children, Classics, Comedy, Family and Music. **Create a query that lists each movie, the film category it is classified in, and the number of times it has been rented out.**

    For this query, you will need 5 tables: `Category`, `Film_Category`, `Inventory`, `Rental and Film`. Your solution should have three columns: Film title, Category name and Count of Rentals.

    - The following table header provides a preview of what the resulting table should look like if you order by category name followed by the film title.
    - ***HINT**:* One way to solve this is to create a count of movies using aggregations, subqueries and Window functions.

- **SOLUTION**

    ```sql
    WITH rented_films AS (
    SELECT fi.title AS movie_title,
        ca.name AS film_category
    FROM category ca
    INNER JOIN film_category fi_ca ON fi_ca.category_id = ca.category_id
    INNER JOIN film fi ON fi.film_id = fi_ca.film_id
    INNER JOIN inventory inv ON inv.film_id = fi.film_id
    INNER JOIN rental re ON re.inventory_id = inv.inventory_id
    WHERE ca.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
    )

    SELECT DISTINCT movie_title, film_category,
        COUNT(film_category) AS rented_count
    FROM rented_films
    GROUP BY DISTINCT(movie_title, film_category)
    ORDER BY rented_count DESC; 
    ```

It could also be interesting to get wich of these genres are the most popular
- **SOLUTION**

    ```sql
    WITH rented_films AS (
    SELECT fi.title AS movie_title,
        ca.name AS film_category
    FROM category ca
    INNER JOIN film_category fi_ca ON fi_ca.category_id = ca.category_id
    INNER JOIN film fi ON fi.film_id = fi_ca.film_id
    INNER JOIN inventory inv ON inv.film_id = fi.film_id
    INNER JOIN rental re ON re.inventory_id = inv.inventory_id
    WHERE ca.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
    )

    SELECT DISTINCT(film_category),
        COUNT(film_category) OVER (PARTITION BY film_category) AS count_category
    FROM rented_films
    ORDER BY 2 DESC;
    ```