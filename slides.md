---
title: Wprowadzenie do JDBC

---

### Wprowadzenie do JDBC
    
![jdbc](assets/intro.png)
---

# **<span style="color:salmon">JDBC</span>** 

### **<span style="color:salmon">J</span>ava <span style="color:salmon">D</span>ata<span style="color:salmon">B</span>ase <span style="color:salmon">C</span>onnectivity** 

**JDBC** to interfejs programowania opracowany w 1996 r, umożliwiający aplikacjom napisanym w języku **Java** porozumiewanie się z bazami danych za pomocą języka **SQL**.
Przy pomocy **JDBC** możemy tworzyć tak zwaną warstwę trwałości (*ang. persistence layer*).

---

##### Architektura wartwowa

Termin *warstwa trwałości* pochodzi z pojęcia **architektury n-wartwowej** (*ang. three-tier architecture*), gdzie najczęściej warstwy są trzy:

![architektura wartwowa](assets/layers.png)

---

**Driver** to interfejs odpowiadający za warstwę bezpośrednio kontaktującą się z bazą danych.
Dla odmiennych silników bazodanowych mogą istnieć różne implementacje sterownika.

![Driver](assets/JDBC.png)

---

**Connection** to klasa reprezentująca połączenie do bazy danych.

Obiekt **Connection** możemy tworzyć za pomocą klasy **DriverManager**:

```java
Connection c = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/test"||1||,
    "user"||2||, 
    "password"||2||
);
```

||1|| Jako pierwszy parametr metody **getConnection** należy podać łańcuch pozwalający zidentyfikować bazę, z którą ma zostać nawiązane połączenie (*ang. jdbc string*). =>
||2|| Kolejnymi argumentami są nazwa użytkownika i hasło. =>

---

Łańcuch znaków **jdbc string** jest przekazywany do drivera i pozwala mu zlokalizować i nawiązać połaczenie z bazą danych.

Format łańcucha może różnic się w zależności od docelowej bazy danych, ale zawsze zaczyna się od nazwy protokołu **jdbc:**.

Dla bazy danych **MySQL** łańcuch ma następujący format:

**protokół:<span style="color:orange">silnik</span>://<span style="color:purple">host</span>:<span style="color:aqua">port</span>/<span style="color:pink">baza</span>** 

**jdbc:<span style="color:orange">mysql</span>://<span style="color:purple">192.168.33.40</span>:<span style="color:aqua">3306</span>/<span style="color:pink">szkola</span>** 

---

Do tworzenia obiektów **Connection** możemy użyć też interfejsu **DataSource**. Dla konkretnych silników bazodanowych będą istnieć konkretne implementacje **DataSource**,
na przykład **MysqlDatasource** dla bazy danych **MySQL**:

```java
MysqlDataSource dataSource = new MysqlDataSource(); ||1||
dataSource.setUser("username");
dataSource.setPassword("password");
dataSource.setUrl("jdbc:mysql://localhost:3306/test");
```

```java
Connection c = datasource.getConnection(); ||2||
```

||1|| Po stworzeniu **dataSource** musimy również podać nazwę użytkownika, hasło oraz łańcuch pozwalający połączyć się z bazą danych. =>
||2|| Po stworzeniu **dataSource** możemy wielokrotnie go wykorzystywać, aby tworzyć obiekty **Connection**. =>

Note: HikariCP, c3p0. Zadania
---
```java
try {
  Class.forName("com.mysql.Driver"); ||1||
  String url = "jdbc:mysql://localhost:3306/test";
  Connection c = DriverManager.getConnection(url, "user", "password");
  Statement s = c.createStatement(); ||2||
  ResultSet rs = s.executeQuery("SELECT name,salary FROM employees"); ||3||
  while (rs.next()) { ||4||
    System.out.println(rs.getString("name") + ": " + rs.getInt("salary")); ||5||
  }
} catch(ClassNotFoundException|SqlException e) { ||6||
  e.printStackTrace();
} finally { ||7||
  rs.close();
  s.close();
  c.close();    
}
```
||1|| Bezpośrednie załadowanie klasy sterownika jest potrzebne tylko w starszych wersjach **JDBC** (poniżej wersji **4.1**). =>
||2|| Po stworzeniu **Connection** możemy za pomoca jego pomocą stworzyć obiekt **Statement**. =>
||3|| Do metody **executeQuery** przekazujemy jako argument zapytanie **SQL**, która ma zostać wykonane na bazie banych. 
 W tym przypadku pobieramy kolumny **name** oraz **salary** z tabeli **employees**. Jako zwracaną wartość otrzymujemy obiekt **ResultSet**, przechowujący wyniki zwrócone przez zapytanie.=>
||4|| Wywołując metodę **next** przesuwamy wewnęrzny wskaźnik **ResultSet** na pozycję kolejnych wierszy. Jeżeli wskaźnik przesunie się na kolejne miejsce, to metoda **next** zwróci **true**, a gdy dotarliśmy do końca wyników, to zwróci **false**. => 
||5|| Używająć metod **get\*** obiektu **ResultSet** możemy pobierać wartości kolumn z wierszy. =>
||6|| Przechwytujemy wyjątki **ClassNotFoundException**, który może zostać wyrzucony przez metodę **Class.forName** oraz  **SqlException**, który
może zostać zgłoszony w przypadku, gdy nie powiedzie się zapytanie do bazy danych. =>
||7|| W bloku **finally** musimy pozamykać wszystkie zasoby, czyli **ResultSet**, **Statement** oraz **Connection**.

---

Użycie bloku **try-with-resources** uprasza strukturę kodu:

```java
String url = "jdbc:mysql://localhost:3306/test";
try( ||1||
  Connection c = DriverManager.getConnection(url, "user", "password");
  Statement s = c.createStatement();
  ResultSet rs = s.executeQuery("SELECT name,salary FROM employees");
) {
  while (rs.next()) {
    System.out.println(rs.getString("name") + ": " + rs.getInt("salary"));
  }
} catch(SQLException e) { ||2||
  e.printStackTrace();
}
```

||1|| Jeżeli otworzymy **Connection**, **Statement** oraz **ResultSet** w nagłówku instrukcji **try**, to na tych obiektach zostanie automatycznie
wywołana metoda **close** w momencie zakończenie wywoływania instrukcji w bloku. Z tego powodu nie musimy już ręcznie dodawać bloku **finally** zawierającego
instrukcje zamykające. Taka deklaracja jest możliwa ponieważ **Connection**, **Statement** oraz **ResultSet** implementują interfejs **java.lang.AutoCloseable**.

||2|| W związku z tym, że nie ładujemy bezpośrednio klasy sterowanika za pomocą **Class.forName**, to w bloku **catch** musimy obsłużyć tylko wyjątek **SQLException**. 


---

Najważniejsze metody interfejsu **Connection**:

* **execute** zwraca **true/false** w zależności czy operacje się powiedzie.
* **executeUpdate** zwraca liczbę dodanych/zmienionych/usuniętych wierszy.
* **executeQuery** zwraca obiekt **ResultSet**.

Note: Pierwsza pula ćwiczeń (1 i 2).

---

Podczas wywoływania metod **get\*** następuje zamiana z typu języka **SQL** na typ języka **Java**.


| Typ DB              | Typ Java               | Metoda ResultSet |
|---------------------|------------------------|------------------|
| CHAR, VARCHAR 	  | String                 | getString        |
| NUMERIC, DECIMAL    |	java.math.BigDecimal   | getBigDecimal    |
| BIT, BOOLEAN        |	boolean                | getBoolean       |
| INTEGER             |	int                    | getInt           |
| BIGINT 			  | long                   | getLong          |
| FLOAT, DOUBLE       |	double                 | getDouble        |
| BINARY, VARBINARY   | byte[]                 | getBytes         |
| DATE 	              | java.sql.Date          | getDate          |
| TIME 	              | java.sql.Time          | getTime          |
| TIMESTAMP           | java.sql.Timestamp     | getTimestamp     |


---

##### Prepared statements

**Prepared Statement** pozwalają na użycie **parametrów** w zapytaniu:

```java
String url = "jdbc:mysql://localhost:3306/test";
try( 
  Connection c = datasource.getConnection();
  PreparedStatement ps = c.prepareStatement(
          "SELECT name,salary FROM employees WHERE name = ? OR age = ?" ||1||
  )
) {
  s.setString(1, "Tom%"); ||2||
  s.setInt(2, 20);
  try(ResultSet rs = ps.executeQuery()) {
    while (rs.next()) {
      System.out.println(rs.getString("name"));
      System.out.println(rs.getInt("salary"));
    }
  }
} catch(SQLException e) {
  e.printStackTrace();
}
```

||1|| W zapytaniu SQL w miejsu gdzie chcemy użyć parametrów wstawiamy znak **?**. =>
||2|| Używając metod **set\*** ustawiamy wartości parametrów. Jako pierwszy argument podajemy pozycję parametru,
a jako drugi wartość.

@! Indeksowanie parametów zaczyna się od **1**. !@ 

---

Do umieszczania w zapytaniu danych pochodzących od klienta, powinniśmy używać **parametrów zapytania**:

```java
"SELECT name, salary FROM employee WHERE name = ?"; !!!OK!!!
```

```java
"SELECT name, salary FROM employee WHERE name = :name"; !!!OK!!!
```

```java
"SELECT name, salary FROM employee WHERE name = '" + name + "'"; !!!NOK!!!
```
@! Używanie niesprawdzonych danych w zapytaniu poprzez łączenie łańcuchów znaków naraża nas na **SQL Injection**. Dane otrzymane z nieznanych źródeł należy ustawiać za pomocą **parametrów zapytania**. !@

---

##### Transakcje

Domyślnie **Connection** działą w trybie **auto-commit**, co oznacza, że każde pojedyncze zapytanie wykonywane jest jako osobna transakcja.

Żeby wykonywać więcej działań w obrębie jednej transakcji ustawiamy **connection.setAutoCommit(false)**, a następnie po wykonaniu dowolnej ilości zapytań możemy wywołać:

* Metodę **commit**, żeby zatwierdzić transakcję.
* Metodę **rollback**, żeby wycofać transakcję.
* Metodę **setSavepoint** żeby stworzyć punkt kontrolny.

---

```java
String url = "jdbc:mysql://localhost:3306/test";
try( 
  Connection c = DriverManager.getConnection(url, "user", "password");
  Statement s = c.createStatement()  
) {
  c.setAutoCommit(false); ||1||
  s.executeUpdate( "UPDATE user SET salary = 5000 WHERE id = 1");
  s.executeUpdate( "UPDATE user SET salary = 6000 WHERE id = 2");
  c.commit(); ||2||
} catch(SQLException e) {
  c.rollback(); ||3||
  e.printStackTrace();
}
```

||1|| Ustawiamy transakcję w tryb ręcznego zatwierdzania. =>
||2|| Po wykonaniu dwóch zapytań zatwierdzamy transakcję. =>
||3|| W przypadku wystąpienia wyjątku możemy wycofać transakcję przy pomocy **rollback**.

---

Możemy wywołać **rollback** aby wycofać całkowicie transakcję, albo **rollback** podając jako parametr *savepoint*,
tak aby wycofać tylko do miejsca wykonania zapisu punktu kontrolnego.

```java
String url = "jdbc:mysql://localhost:3306/test";
try(
    Connection c = DriverManager.getConnection(url, "user", "password");
    Statement s = c.createStatement()
) {
    c.setAutoCommit(false);
    s.executeUpdate( "UPDATE user SET salary = 5000 WHERE id = 1");
    Savepoint sp = c.setSavepoint("sp"); ||1||
    s.executeUpdate( "UPDATE user SET salary = 6000 WHERE id = 2");
    int max = s.executeQuery("SELECT max(salary) FROM user").getInt(1);
    if(max > 10000) {
        c.rollback(sp); ||2||
    } else if (max < 8000) {
        c.rollback(); ||3||
    } else {
        c.commit(); ||4||
    }
} catch(SQLException e) {
    e.printStackTrace();
    c.rollback();
}
```

||1|| Tworzymy punkt kontrolny po wykonaniu pierwszej instrukcji. =>
||2|| Wycofujemy tylko drugą instrukcję. =>
||3|| Wycofujemy wszystkie instrukcje od rozpoczęcie transakcji. =>
||4|| Potwierdzamy transakcję.

---
##### Wywoływanie procedur

Żeby wywołać procedurę na serwerze bazodanowym używamy obiektu typu **CallableStatement**:

```java
try (
    Connection c = DriverManager.getConnection(url, "user", "password");
    CallableStatement cs = c.prepareCall("call my_procedure(?, ?)"); ||1||
) {
    cs.setLong(1, id); ||2||
    cs.setLong(1, name);
    cs.executeQuery();
} catch (SQLException exception) {
    e.printStackTrace();
}
```

||1|| Metoda **my_procedure** musi istnieć w bazie danych. =>
||2|| Za pomocą metod **set\*** ustawiamy parametry procedury.

---
##### Grupowanie zapytań

Wiele zapytań możemy grupować w partie (*ang. batch*):

```java
Statement stmt = conn.createStatement();
stmt.addBatch("INSERT INTO Employees (id, name) VALUES(200,'Staś')"); ||1||
stmt.addBatch("INSERT INTO Employees (id, name) VALUES(201,'Krzyś')");
stmt.addBatch("INSERT INTO Employees (id, name) VALUES(202,'Zbyś')");

int[] count = stmt.executeBatch(); ||2||
```

||1|| Za pomocą **addBatch** dodajemy kolejne instrukcje do partii.

||2|| Wywołanie **executeBatch** powoduje wykonanie wszystkich instrukcji w partii. Metoda zwraca tablicę wartości **int** oznaczającą ilość
wierszy zmodyfikowaną przez każdą z instrukcji.

---

### JDBC TEMPLATE

**JdbcTemplate** jest biblioteką, która ułatwia korzystanie z **JDBC**.

---

Aby skorzystać z biblioteki **JdbcTempate** musi dodać dodaktową zależność do projektu, na przykład poprzez dodanie **dependency** do 
pliku **pom.xml**:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.1.6.RELEASE</version>
</dependency>
```

---

Aby pobrać dane z bazy danych możemy skorzystać z metody **query**:

```java
JdbcTemplate jdbc = new JdbcTemplate(dataSource); ||1||
Employee employee = jdbc.query(
  "SELECT id, name, salary FROM employee WHERE id = ?", ||2||
  new Object[] {id}, ||3||
  new EmployeesExtractor() ||4||
);
```

||1|| Tworzymy instancję **JdbcTemplate** przekazując jako parametr **datasource**. =>
||2|| Wywołując metodę **query** jako pierwszy parametr przekazujemy zapytanie **SQL**. =>
||3|| Kolejnym parametrem jest tablica, która ma zawierać wartości dla wszystkich **parametrów zapytania**. =>
||4|| Ostatnim parametrem jest obiekt klasy dziedziczącej po interfejsie **ResultSetExtractor**. 

---

```java
public class EmployeesExtractor implements ResultSetExtractor<Employee> {

  @Override
  public Employee extractData(ResultSet rs) throws SQLException { ||1||
    if (rs.next()) {
      return new Employee( ||2||
          rs.getLong("id"),
          rs.getString("name"),
          rs.getInt("salary")
      );
    } else {
      throw IllegalStateException("No student found!"); ||3||
    }
  }
}
```

||1|| Implementujac **ResultSetExtractor** musimy przesłonić metodę **extractData**, gdzie jako argument otrzymujemy **ResultSet** z wynikiem zapytania. =>
||2|| Zwracamy instację klasy **Employee** pobierając wszystkie potrzebne dane do jej stworzenia z **ResultSet**. =>
||3|| W przypadku, gdy **ResultSet** jest pusty zgłaszamy wyjątek.


---

Używając klasy **NamedParameterJdbcTemplate** możemy skorzystać z **nazwanych parametrów zapytania** (*ang. named query parameters*).

```java
NamedParameterJdbcTemplate jdbc = new NamedParameterJdbcTemplate(dataSource); ||1||
jdbc.query(
  "SELECT name, salary FROM employee " +
  "WHERE name LIKE :query OR department LIKE :query OR id = :id", ||2||
  new MapSqlParameterSource("query", query + "%").addValue("id", 100), ||3||
  new StudentsResultSetExtractor()
);

```

||1|| Anologicznie jak przy tworzeniu **JdbcTemplate** przekazujemy **dataSource** jako argument konstruktora **NamedParameterJdbcTemplate**. =>
||2|| Parametry zapytania zapisujemy jako **:nazwa**. Jeżeli chcemy parokrotnie użyć tej samej wartości, to możemy powtórzyć parametr. =>
||3|| Aby przekazać wartości parametrów używamy **MapSqlParameterSource** podając jako argumenty konstruktora nazwę parametru i jego wartość.
Kolejne parametry podajemy używać metody **addValue**.

---

Przy pomocy **JdbcTemplate** możemy również wykonywać zapytania zgrupowane w **Batch**:

```java
int[] modified||1|| = jdbcTemplate.batchUpdate(
"UPDATE employees SET salary = ? WHERE salary = ?",
new BatchPreparedStatementSetter() { ||2||
  public void setValues(PreparedStatement ps, int i) throws SQLException {
    ps.setInt(1, newSalaries.get(i)); ||3||
    ps.setInt(2, oldSalaries.get(i));
  }

  public int getBatchSize() { ||4||
    return newSalaries.size();
  }
});
```
||1|| Metoda **batchUpdate** zwraca tablicę zawierającą w każdej komórce liczbę wierszy zmodyfikowanych przez kolejne instrukcje.

||2|| Tworzymy anonimową klasę implementującą interfejs **BatchPreparedStatementSetter**.

||3|| w metodzie **setValues** przygotowujemy kolejne **PreparedStatement** ustawiając wartości z kolekcji **newSalaries** oraz **oldSalaries** korzystająć
z wartości licznika przekazanej jako drugi argument.

||4|| W metodzie **getBatchSize** zwracamy wielkość paczki.

---

<div class="icon-line">
    <img alt="github" src="assets/github.svg"/>&nbsp;&nbsp;&nbsp;<a href="https://github.com/katlasik">https://github.com/katlasik</a>
</div>
<div class="icon-line">
     <img alt="mail" src="assets/mail.png"/>&nbsp;&nbsp;&nbsp;<a href="mailto:krzysztof.atlasik@pm.me">krzysztof.atlasik@pm.me</a>
</div>
