Set values using SQL functions/stored procedures
================================================

Let's say you want to set the value of some fields as the output of some custom SQL functions or stored procedures.

This can be achieved by:

#. Modifying the table to allow the columns we want to be set by the SQL functions as nullable.
#. Creating an insert/update trigger on the table that calls your SQL function and sets the output values in the output
   columns.
#. Making your mutation requests without setting the SQL function output columns.

**For example**, say we have a table ``sql_function_table`` with columns ``input`` and ``output`` and we would like
to set the value of the ``output`` column as the uppercased value of the string received in ``input`` field.

1) Modify the table
-------------------

Modify the table ``sql_function_table`` and make its ``output`` column nullable.

Head to ``Data -> sql_function_table -> Modify``

.. image:: ../../../img/graphql/manual/schema/modify-sql-fn-table.png

2) Create a trigger
-------------------

The below SQL defines a ``trigger`` which will simply uppercase the value passed in the ``input`` field and set it to
the ``output`` field whenever an insert or update is made to the ``sql_function_table``.

.. code-block:: plpgsql

   CREATE FUNCTION test_func() RETURNS trigger AS $emp_stamp$
         BEGIN
             NEW.output := UPPER(NEW.input);
             RETURN NEW;
         END;
     $emp_stamp$ LANGUAGE plpgsql;

     CREATE TRIGGER test_trigger BEFORE INSERT OR UPDATE ON sql_function_table
         FOR EACH ROW EXECUTE PROCEDURE test_func();

Head to ``Data -> SQL`` and run the above SQL:

.. image:: ../../../img/graphql/manual/schema/create-trigger.png

3) Run an insert mutation
-------------------------

Run a mutation to insert an object with (input = "yabba dabba doo!", output=null) and you'll see the output
value (output="YABBA DABBA DOO!") will be set automatically.

.. graphiql::
  :view_only:
  :query:
    mutation {
      insert_sql_function_table (
        objects: [
          {input: "yabba dabba doo!"}
        ]
      ) {
        returning {
          input
          output
        }
      }
    }
  :response:
    {
      "data": {
        "insert_sql_function_table": {
          "returning": [
            {
              "input": "yabba dabba doo!",
              "output": "YABBA DABBA DOO!"
            }
          ]
        }
      }
    }
