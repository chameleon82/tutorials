## Convert java.sql.Date to/from LocalDate and Timestamp

```java
// Instantiate with ISO-8601 format
java.sql.Date sqlDate = java.sql.Date.valueOf("2019-01-29");

// Convert from sql to local
java.time.LocalDate localDate = sqlDate.toLocalDate();

// Convert from local to sql
java.sql.Date newSqlDate = java.sql.Date.valueOf(localDate);

// To timestamp in epoch format (Be careful with timezone)
long epoch = localDate.atStartOfDay().toEpochSecond(ZoneOffset.UTC);
// 1548720000L

// From sql through LocalDate
// java.sql.Date.valueOf("2019-01-29").toLocalDate.atStartOfDay().toEpochSecond(ZoneOffset.UTC);
// 1548720000L

// To LocalDate from epoch
LocalDateTime.ofEpochSecond(new Timestamp(1548720000L).getTime,0, ZoneOffset.UTC).toLocalDate

```
