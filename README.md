# rust_fastdb

`rust_fastdb` is a fast, embedded database engine written in Rust with Python bindings using PyO3.

---

## 🚀 Features

- ✅ High-performance data access
- 🐍 Easy-to-use Python interface
- 🦀 Built with Rust (safe concurrency and speed)
- 💾 Supports PostgreSQL, MySQL, and SQLite
- 📦 Simple installation with `pip install`

---

## 📦 Installation

```bash
pip install rust_fastdb
```

---

## 📦 For Customation (Comming Soon)
 - Clone This Repository to your local machine
 - Makesure you already install rust dan python
 - Recomended to run using Virtual Environment 
 


---

## 📝 Changelog
### v0.1.0

- ✅ **Initial Release**
- 🔄 Added support for querying data from databases
- 💽 Supported databases:
  - PostgreSQL
  - MySQL
  - SQLite

---

## 🧪 Example
```bash
  db_url = "postgres://<username>:<password>@<host>:<port>/<database>" //you can use env
  user_data = await rust_fastdb.run_manual_query_with_params(
            db_url,
            "SELECT id::text AS id, fullname FROM users;", #if your id using UUID please convert to text otherwise it will getting an error result
            [],      # Leave empty if no ? parameters
            False    # use_cache 
        )
```