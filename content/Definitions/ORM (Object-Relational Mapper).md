This is a framework where the structure of your classes in the code (your objects) maps directly to the data structure used in your relational database.

This should mean that you don't really need to write any SQL. You just create a base DB class (from which all objects inherit, or implement, or however you want to do it in your language) which has the necessary .load() and .save() and .update() and other functions that you might in any CRUD application. 

All of the columns in that table become an attribute of the class. 