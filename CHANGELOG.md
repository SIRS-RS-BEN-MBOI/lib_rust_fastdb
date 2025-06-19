# 📦 Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/).

---

## [0.1.0] - 2025-06-19

### Added
- ✅ Initial release
- 🔄 Support for executing read-only (SELECT) queries via Rust backend
- 💽 Database engine support:
  - PostgreSQL
  - MySQL
  - SQLite

### Limitations
- 📥 Currently supports **only** reading (GET/SELECT) data
- ❌ No support yet for `INSERT`, `UPDATE`, `DELETE`

---

## 🛣️ Coming Soon

### Planned for [0.2.0]
- ✏️ Write operations: `INSERT`, `UPDATE`, `DELETE`
- 🛡️ Connection pooling for better performance
- 🔐 Encrypted connection string support
- 📊 Query result formatting (JSON/dict output)
- 🧪 Unit tests and async benchmarks
- 📚 More usage examples and documentation
