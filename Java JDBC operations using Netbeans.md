## Setting up a project 
- Open Netbeans
- New Project - Java using Ant - Java Application
- Right click on Libraries under Project tab - Add JAR/Folder
- Add the JDBC [Connector](https://dev.mysql.com/downloads/connector/j/) (.jar file)
- Edit the Source code in the .java file under Source packages under Project tab
![](https://github.com/CyanFroste/JAVA-DB/blob/master/Images/operations.png)

# OPERATIONS

## Printing Query results
```
package jdbctest;
import java.sql.*;

public class Jdbctest {
    private static final String URL = "jdbc:mysql://localhost:3306/test?useLegacyDatetimeCode=false&serverTimezone=UTC"; 
    private static final String USER = "root";
    private static final String PASS = "";
    
    public static void main(String[] args) {
        try{
            Connection con = DriverManager.getConnection(URL, USER, PASS);

            Statement stmt = con.createStatement();

            String sql = "SELECT * FROM jdbctest";

            ResultSet res = stmt.executeQuery(sql);

            while(res.next()){
                System.out.println(res.getString("name")+','+res.getString("location"));
            }        
        }
        catch(SQLException e){
            System.err.println(e);
        }        
    }   
}
```

## Insert
```
package jdbctest;
import java.sql.*;

public class Jdbctest {
    private static final String URL = "jdbc:mysql://localhost:3306/test?useLegacyDatetimeCode=false&serverTimezone=UTC"; 
    private static final String USER = "root";
    private static final String PASS = "";
    
    public static void main(String[] args) {
        try{
            Connection con = DriverManager.getConnection(URL, USER, PASS);
            Statement stmt = con.createStatement();
            String sql = "INSERT INTO jdbctest VALUES (1,'Cyan','Kollam',NULL)";
            stmt.executeUpdate(sql);
            System.out.println("Inserted");
            }
        catch(SQLException e){
            System.err.println(e);
        }        
    }   
}
```

## Update
```
package jdbctest;
import java.sql.*;

public class Jdbctest {
    private static final String URL = "jdbc:mysql://localhost:3306/test?useLegacyDatetimeCode=false&serverTimezone=UTC"; 
    private static final String USER = "root";
    private static final String PASS = "";
    
    public static void main(String[] args) {
        try{
            Connection con = DriverManager.getConnection(URL, USER, PASS);
            Statement stmt = con.createStatement();
            String sql = "UPDATE jdbctest SET location = 'Miami' WHERE ID = 1";
            stmt.executeUpdate(sql);
            System.out.println("Updated");
            }
        catch(SQLException e){
            System.err.println(e);
        }        
    }   
}
```

## Deletion
```
package jdbctest;
import java.sql.*;

public class Jdbctest {
    private static final String URL = "jdbc:mysql://localhost:3306/test?useLegacyDatetimeCode=false&serverTimezone=UTC"; 
    private static final String USER = "root";
    private static final String PASS = "";
    
    public static void main(String[] args) {
        try{
            Connection con = DriverManager.getConnection(URL, USER, PASS);
            Statement stmt = con.createStatement();
            String sql = "DELETE FROM jdbctest WHERE name = 'Cyan'";
            
            int rowsAffected = stmt.executeUpdate(sql);
            
            System.out.println("Rows Affected:" + rowsAffected);
            System.out.println("Deleted");
            }
        catch(SQLException e){
            System.err.println(e);
        }        
    }   
}
```

## Prepared Statements
Instead of Hardcoding the SQL, Use this:
```
package jdbctest;
import java.sql.*;

public class Jdbctest {
    private static final String URL = "jdbc:mysql://localhost:3306/test?useLegacyDatetimeCode=false&serverTimezone=UTC"; 
    private static final String USER = "root";
    private static final String PASS = "";
    
    public static void main(String[] args) throws SQLException {
        
        Connection con = null;
	PreparedStatement stmt = null;
	ResultSet res = null;
        
        try{
            con = DriverManager.getConnection(URL, USER, PASS);
            
            String sql = "SELECT * FROM jdbctest WHERE ID = ? AND name = ?" ;

            stmt = con.prepareStatement(sql);            
            
            stmt.setInt(1,  1);
            stmt.setString(2, "Cyan");
            
            res = stmt.executeQuery();
            
            display(res);
        }        
        catch(SQLException e){
            System.err.println(e);
        }
        finally {
		if (res != null) {
			res.close();
            }
		if (stmt != null) {
            stmt.close();
		    }
		if (con != null) {
			con.close();
		    }
	    }
    }   

    private static void display(ResultSet res) throws SQLException {
		while (res.next()) {
			int ID = res.getInt("ID");
			String name = res.getString("name");
			String location = res.getString("location");			
			System.out.printf("%d, %s, %s" , ID, name, location);
		}
	}
}
```

## JDBC Transactions
Save all the made changes or None at all

```
package jdbctest;

import java.sql.*;
import java.util.Scanner;

public class Jdbctest {
    private static final String URL = "jdbc:mysql://localhost:3306/test?useLegacyDatetimeCode=false&serverTimezone=UTC"; 
    private static final String USER = "root";
    private static final String PASS = "";
    
    public static void main(String[] args) throws SQLException {
        
        Connection con = null;
	Statement stmt = null;
	ResultSet res = null;
        
        try{
            con = DriverManager.getConnection(URL, USER, PASS);
            
                // Turn off auto commit
		con.setAutoCommit(false);

		// Show salaries BEFORE
		System.out.println("Salaries BEFORE\n");
		showSalaries(con, "HR");
		showSalaries(con, "Engineering");

		// Transaction Step 1: Delete all HR employees
		stmt = con.createStatement();
		stmt.executeUpdate("delete from employees where dept='HR'");

		// Transaction Step 2: Set salaries to 300000 for all Engineering
		stmt.executeUpdate("update employees set salary=300000 where dept='Engineering'");

		System.out.println("\n>> Transaction steps are ready.\n");

		// Ask user if it is okay to save
		boolean ok = save();

		if (ok) {
			// store in database
			con.commit();
			System.out.println("\n>> Transaction COMMITTED.\n");
		} else {
			// discard
			con.rollback();
			System.out.println("\n>> Transaction ROLLED BACK.\n");
		}
                // Show salaries AFTER
		System.out.println("Salaries AFTER\n");
		showSalaries(con, "HR");
		showSalaries(con, "Engineering");

	} catch (SQLException e) {
		System.err.println(e);
	} finally {
		close(con, stmt, null);
	}
}
    
    private static boolean save() {
        String input;
        try (Scanner scanner = new Scanner(System.in)) {
            System.out.println("Is it okay to save?  yes/no: ");
            input = scanner.nextLine();
        }

            return input.equalsIgnoreCase("yes");
	}

    private static void showSalaries(Connection con, String theDepartment) throws SQLException {
	PreparedStatement stmt = null;
	ResultSet res = null;

	System.out.println("Show Salaries for Department: " + theDepartment);

	try {
		// Prepare statement
		stmt = con.prepareStatement("select * from employees where dept=?");

		stmt.setString(1, theDepartment);

		// Execute SQL query
		res = stmt.executeQuery();

		// Process result set
		while (res.next()) {
			int ID = res.getInt("ID");
			String name = res.getString("name");
			String dept = res.getString("dept");
			int salary = res.getInt("salary");

			System.out.printf("%d, %s, %s, %d\n", ID, name, dept, salary);
		}

		System.out.println();
	} catch (SQLException e) {
		System.err.println(e);
	} finally {
		close(stmt, res);
	}

}

    private static void close(Connection con, Statement stmt, ResultSet res) throws SQLException {
	if (res != null) {
		res.close();
	}

	if (stmt != null) {
		stmt.close();
	}

	if (con != null) {
		con.close();
	}
}
    
    private static void close(Statement stmt, ResultSet res) throws SQLException {

	close(null, stmt, res);
    }
}
```


