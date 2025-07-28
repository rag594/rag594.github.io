+++
title = "UUID vs SnowflakeId vs ULID in mysql for secondary index"
date = "2022-10-20"
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
[taxonomies]
tags = ["innodb", "mysql", "ulid", "snowflakeId", "uuid"]
+++

Did some tinkering on how the data is stored in pages when UUID, SnowflakeId and ULID are stored as secondary indexes.

## UUID

### Schema

```sql
CREATE TABLE `users_uuid` (
  `id` int NOT NULL AUTO_INCREMENT,
  `uuid` varchar(36) NOT NULL,
  `name` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_uuid` (`uuid`)
) ENGINE=InnoDB AUTO_INCREMENT=2000001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

### Page layout

![Page Layout 1](/uuid_page1.png)

![Page Layout 2](/uuid_page2.png)

![Page Layout 3](/uuid_page3.png)


## SnowflakeId

### Schema

```sql
CREATE TABLE `users_snowflake` (
  `id` int NOT NULL AUTO_INCREMENT,
  `snowflake_id` bigint unsigned NOT NULL,
  `name` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_snowflake_id` (`snowflake_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2000001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

### Page Layout

![Page Layout 1](/snowflake_page1.png)

![Page Layout 2](/snowflake_page2.png)


## ULID

### Schema

```sql
CREATE TABLE `users_ulid` (
  `id` int NOT NULL AUTO_INCREMENT,
  `ulid` varchar(26) NOT NULL,
  `name` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_ulid` (`ulid`)
) ENGINE=InnoDB AUTO_INCREMENT=1000001 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```


### Page Layout

![Page Layout 1](/ulid_page1.png)

![Page Layout 2](/ulid_page2.png)


## Script to generate Millions of Ids

```go
package main

import (
	"database/sql"
	"flag"
	"fmt"
	"log"
	"math/rand"
	"time"

	"github.com/bwmarrin/snowflake"
	_ "github.com/go-sql-driver/mysql"
	"github.com/google/uuid"
	"github.com/oklog/ulid/v2"
)

// Snowflake ID generator (naive)
func generateSnowflakeID() uint64 {
	// Create a new Node with a Node number of 1
	node, err := snowflake.NewNode(rand.Int63n(1023))
	if err != nil {
		fmt.Println(err)
	}
	return uint64(node.Generate().Int64())
}

// ULID generator
func generateULID() string {
	entropy := ulid.Monotonic(rand.New(rand.NewSource(time.Now().UnixNano())), 0)
	id := ulid.MustNew(ulid.Timestamp(time.Now()), entropy)
	return id.String()
}

func insertUUIDs(db *sql.DB, total int) {
	stmt, err := db.Prepare("INSERT INTO users_uuid (uuid, name) VALUES (?, ?)")
	if err != nil {
		log.Fatal(err)
	}
	defer stmt.Close()

	for i := 0; i < total; i++ {
		id := uuid.New().String()
		name := fmt.Sprintf("User_%d", i)
		if _, err := stmt.Exec(id, name); err != nil {
			log.Fatal(err)
		}
		if i%10000 == 0 {
			log.Printf("Inserted %d UUID rows...", i)
		}
	}
}

func insertSnowflakeIDs(db *sql.DB, total int) {
	stmt, err := db.Prepare("INSERT INTO users_snowflake (snowflake_id, name) VALUES (?, ?)")
	if err != nil {
		log.Fatal(err)
	}
	defer stmt.Close()

	for i := 0; i < total; i++ {
		id := generateSnowflakeID()
		name := fmt.Sprintf("User_%d", i)
		if _, err := stmt.Exec(id, name); err != nil {
			log.Fatal(err)
		}
		if i%10000 == 0 {
			log.Printf("Inserted %d Snowflake rows...", i)
		}
	}
}

func insertULIDs(db *sql.DB, total int) {
	stmt, err := db.Prepare("INSERT INTO users_ulid (ulid, name) VALUES (?, ?)")
	if err != nil {
		log.Fatal(err)
	}
	defer stmt.Close()

	for i := 0; i < total; i++ {
		id := generateULID()
		name := fmt.Sprintf("User_%d", i)
		if _, err := stmt.Exec(id, name); err != nil {
			log.Fatal(err)
		}
		if i%10000 == 0 {
			log.Printf("Inserted %d ULID rows...", i)
		}
	}
}

func main() {
	var mode string
	var count int
	flag.StringVar(&mode, "mode", "uuid", "Mode: uuid | snowflake | ulid")
	flag.IntVar(&count, "count", 1000000, "Number of rows to insert")
	flag.Parse()

	dsn := "rag594:raghav@tcp(127.0.0.1:3306)/test_ids?parseTime=true"
	db, err := sql.Open("mysql", dsn)
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	start := time.Now()

	switch mode {
	case "uuid":
		insertUUIDs(db, count)
	case "snowflake":
		insertSnowflakeIDs(db, count)
	case "ulid":
		insertULIDs(db, count)
	default:
		log.Fatalf("Unknown mode: %s", mode)
	}

	log.Printf("Done inserting %d rows using %s. Took: %s\n", count, mode, time.Since(start))
}
```
