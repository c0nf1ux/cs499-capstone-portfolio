# Code Review: Findings

Every claim below points at a real line in a real file. Links go to [engineering-portfolio](https://github.com/c0nf1ux/engineering-portfolio).

---

## 1. ABCU Advising Program (CS-300 — Algorithms & Data Structures)

[`CS300_Project_Two_Heath_Davis.cpp`](https://github.com/c0nf1ux/engineering-portfolio/blob/main/coursework/CS-300_Data-Structures-and-Algorithms/CS300_Project_Two_Heath_Davis.cpp)

A C++ console app that loads a course catalog from CSV into a `std::map<string, Course>` (`courseMap`) and lets the user look courses up. `Course` holds a course number, name, and a `vector<string>` of prerequisites. `loadDataStructure` parses the CSV and builds the map; `printCourseList` leans on the map's automatic key ordering to print an alphanumeric listing for free; `printCourse` does a case-insensitive single-course lookup plus its prerequisites; `displayMenu`/`main` drive a plain text menu around those three operations.

**What's solid:** the file is checked for existence before reading and closed explicitly afterward — one checklist item this code actually gets right.

**What's not:**

- **Read-only, never exercises a write path.** There's a Create and a Read, no Update or Delete. Fine for a catalog browser, but it means there's no evidence the data structure holds up under mutation.
- **Prerequisites have zero validation.** They're stored as plain strings. A CSV listing a prerequisite that doesn't exist, or two courses requiring each other, gets printed as if it were valid — nothing in this file would ever notice.
- **No unit tests.** Correctness was verified by eyeballing console output against a sample-output document, not by anything that runs automatically.
- **The complexity claim is unmeasured.** `std::map` (a balanced binary tree, O(log n) lookup) is the only structure that's ever touched this data. Nothing in the codebase benchmarks it against a hash table, which would be O(1) average case. Right now "O(log n) was the right call" is a textbook claim, not a measured one.
- **Magic numbers in the menu.** `main` branches on raw literals (`case 1`, `case 2`, `case 9` for exit) with no named constant or enum backing any of them.
- **A real bug, not a style gap:** `main` reads the menu choice with `cin` and never checks whether the read succeeded. A non-numeric input puts `cin` into a fail state that's never cleared (`cin.clear()`, `cin.ignore()` appear nowhere in this file) — every loop iteration after that fails identically, so the program just repeats "not a valid option" forever. This is a loop-termination defect, not a hypothetical one; the exact missing lines are identifiable.
- **Duplicated effort.** The CSV-splitting logic here is written from scratch, even though an earlier assignment in the same course already built a reusable CSV parser class.

---

## 2. Secure REST Service (CS-305 — Software Design & Engineering)

Two separate course projects, never actually connected to each other — merging them is itself the planned enhancement.

**[`CRUDController.java`](https://github.com/c0nf1ux/engineering-portfolio/blob/main/coursework/CS-305_Software-Security/CS%20305%20Project%20One%20Code%20Base/rest-service/src/main/java/com/twk/restservice/CRUDController.java)** exposes exactly one endpoint (`/read`) that takes a business name and returns a `CRUD` object built from `DocData`. No create, update, or delete endpoint exists despite the class's name. **[`CRUD.java`](https://github.com/c0nf1ux/engineering-portfolio/blob/main/coursework/CS-305_Software-Security/CS%20305%20Project%20One%20Code%20Base/rest-service/src/main/java/com/twk/restservice/CRUD.java)** is a plain data holder — two fields, named `content` and `content2`, exactly the lazy-naming pattern the course's own review checklist calls out. **[`customer.java`](https://github.com/c0nf1ux/engineering-portfolio/blob/main/coursework/CS-305_Software-Security/CS%20305%20Project%20One%20Code%20Base/rest-service/src/main/java/com/twk/restservice/customer.java)** is dead scaffolding: `showInfo`/`deposit` operate on an account balance that's never initialized, and nothing in the project ever instantiates this class.

**[`SslServerApplication.java`](https://github.com/c0nf1ux/engineering-portfolio/blob/main/coursework/CS-305_Software-Security/CS%20305%20Project%20Two%20Code%20Base/ssl-server_student/src/main/java/com/snhu/sslserver/SslServerApplication.java)** is a standalone 13-line Spring Boot app that boots over TLS and does nothing else — it still carries the course skeleton's original `// FIXME: Add route to enable checksum return of static data`, never implemented. This TLS server and the CRUD service above are two independent Maven projects that happen to live in the same course module; the CRUD service does not actually run behind this TLS boundary.

**The real problems are one layer deeper, in `DocData`:**

- `read_document` opens a raw JDBC connection with the credentials **hardcoded directly in the method**: `localhost:3306`, database `test`, username `root`, password `root` — a second hardcoded credential in the same artifact, and a textbook default-admin pair that should never appear in source.
- That same method catches its `SQLException` and does nothing but `e.printStackTrace()` — not error handling, and exactly the "don't leak internal detail" problem the checklist warns about, sitting right next to an untouched `// TODO Auto-generated catch block` left by the IDE scaffold.
- `read_document` is **never called from anywhere.** `CRUDController` only constructs a bare `DocData()` and calls `.toString()` on it — and `DocData` never defines its own `toString`, so that call falls through to Java's default `Object.toString()`. The one endpoint in this artifact that's supposed to return something returns a class name and a hash code. Not a hypothetical failure — that's what the code does if you run it.

**Everything else the checklist surfaces:** no input validation on the `business_name` parameter (it's declared and never used), no role-based access control, an OWASP Dependency Check plugin wired into the Maven build but not enforced (a known-vulnerable dependency generates a report, doesn't fail the build), and the TLS boundary that's supposed to protect the service never actually touches it.

---

## 3. Grazioso Salvare Animal Rescue Dashboard (CS-340 — Databases)

**[`CRUD_Python_Module.py`](https://github.com/c0nf1ux/engineering-portfolio/blob/main/coursework/CS-340_Client-Server-Development/CRUD_Python_Module.py)** — an `AnimalShelter` class wrapping PyMongo over ~10,000 Austin Animal Center intake/outcome records, consumed by **[`ProjectTwoDashboard.ipynb`](https://github.com/c0nf1ux/engineering-portfolio/blob/main/coursework/CS-340_Client-Server-Development/Project%202%20Heath%20Davis/ProjectTwoDashboard.ipynb)**, a Dash/Plotly app doing filtering, a breed-distribution chart, and a geographic map for Grazioso Salvare's rescue-dog identification work. `create`/`read` both follow a consistent try/except-with-clear-return-value pattern.

- **A plaintext password committed to source:** `PASS = 'MySecurePassword123'`, alongside `HOST`, `PORT`, `DB`, and `COL` all hardcoded right next to it. Fixing the one field that looks worst isn't enough — the whole connection config needs externalizing.
- **The same password is hardcoded a second time, in a completely different file.** The dashboard notebook that imports this class has the identical literal string typed into one of its own cells, passed in as a constructor argument. The credential doesn't live in one place that can be rotated once — it lives in two files kept in sync only by memory.
- **Those two files have already drifted.** The notebook constructs `AnimalShelter` by passing a username and password as two arguments; the class shown here takes zero constructor arguments — the credentials are hardcoded inside it instead. The notebook is calling a constructor signature that no longer matches what's on screen. Exactly the risk the checklist's duplicated-code question is getting at: when the same logic exists twice, nobody can guarantee a fix in one place reaches the other.
- **Incomplete CRUD.** Only create and read exist — no update, no delete.
- **No indexing** on the fields the dashboard actually filters and sorts on (flagged previously in the project's own README, never acted on). **No logging** — no audit trail for who read or wrote what, when.
- A minor one: `create` raises its own exception just to immediately catch that same exception one line later in the same `try` block — extra structure for no reason; it could return `false` directly.

---

**Next:** [PORTFOLIO-DECISIONS.md](PORTFOLIO-DECISIONS.md) — what I'd actually do about each of these, and why.
