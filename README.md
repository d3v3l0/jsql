# jsql

Experimental DSL for SQL generation for next version of clojure.java.jdbc.

## Usage

Generate SQL and parameters that can be use in **with-query-results** style expression:
```clojure
(select * :person)
;;=> ("SELECT * FROM person")
(select [:id :name] :person
  (where {:email "user@domain.com"}))
;;=> ("SELECT id, name FROM person WHERE email = ?" "user@domain.com")
(select [{:p.id :userid} :p.name :a.city] {:person :p}
  (join {:address :a} {:p.id :a.personid}
  (where {:p.email "user@domain.com"}))
;;=> ("SELECT userid, p.name FROM person p JOIN address a ON p.id = a.personid
;;     WHERE p.email = ?" "user@domain.com")
(update :person {:status "active"})
;;=> ("UPDATE person SET status = ?" "active")
(update :person {:status "suspended"}
  (where {:country "NG"}))
;;=> ("UPDATE person SET status = ? WHERE country = ?" "suspended" "NG")
(delete :person (where {:id 42}))
;;=> ("DELETE FROM person WHERE id = ?" 42)
```
**select** expects a *column-spec*, a *table-spec*, optional *join-clauses* (as strings), an optional *where-clause* (sequence of SQL conditions as a string followed by parameters). It returns a sequence whose first element is a string containing a SQL SELECT statement and whose remaining elements are the values to be substituted for parameters (**?**) in that string.

A *column-spec* may be \*, a single *column* or a sequence of *columns*. A *column* may be a string or keyword, or a map (of a single key to a single value, that specifies a column alias).

A *table-spec* may be a string or a keyword, or a map (of a single key to a single value, that specifies a table alias).

A *join-clause* is just a string containing a SQL JOIN clause. It can be generated by the **join** function.

A *where-clause* is a sequence whose first element is a string containing SQL conditions and whose remaining elements are the values to be substituted for parameters (**?**) in that string. It can be generated by the **where** function.

**delete** expects a *table-spec* and a *where-clause* (which is not optional). It returns a sequence whose first element is a string containing a SQL DELETE statement and whose remaining elements are the values to be substituted for parameters (**?**) in that string.

**insert** has yet to be designed.

**join** expects a *table-spec* and a *join-map* which is used to generate the ON clause. It returns a string containing a SQL JOIN/ON clause.

A *join-map* is a map whose keys and values represent columns in the two tables being joined.

**where** expects a *value-map* which is used to generate a string that contains the conditional part of a WHERE clause and the values to be substituted for parameters (**?**) within that string. It returns a sequence whose first element is the string and whose remaining elements are the parameter values.

**update** expects a *table-spec*, an *update-map* and an optional *where-clause*. It returns a sequence whose first element is a string containing a SQL UPDATE statement and whose remaining elements are the values to be substituted for parameters (**?**) in that string.

An *update-map* is a map whose keys represent columns to be set to the corresponding values.

All functions that generate SQL may have an optional **:entities** keyword argument (after the specified arguments) which specifies an identifier naming convention, e.g., **(quoted \\`)** which would cause SQL entities to be quoted with **\`** (**:id** would become **\`id\`**). Other built-in naming conventions as **as-is** and **lower-case**.

The **entities** macro can be used to apply an identifier naming convention to a DSL expression. It expects a function representing the naming convention and a DSL expression. It post-walks the DSL expression and inserts the **:entities** keyword argument and naming convention at the end of each expression.

The remaining parts of the DSL are the high level parts that will execute SQL and process results. Current thinking is along these lines:
```clojure
(query db sql-params :result-set rs-fn :row row-fn)
;;=> runs a SELECT SQL statement
;;   applies row-fn to each row returned (default: identity)
;;   applies rs-fn to the whole result set (default: doall)

(execute! db sql-params)
;;=> runs a non-SELECT SQL statement and returns the update counts, if appropiate

;; I'm still thinking about how inserts should work...
(insert! db :table value-maps)
(insert! db :table col-seq value-seqs)

(update! db :table update-map where-clause)
;;=> (execute! db (update :table update-map where-clause))

(delete! db :table where-clause)
;;=> (execute! db (delete :table where-clause)
```

## License

Copyright © 2012 Sean Corfield

Distributed under the Eclipse Public License, the same as Clojure.
