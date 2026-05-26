# Evidence SQL Injection fixed
Running semgrep on *juice-shop/routes/login.ts*
We notice that there's a problem that could affect all the web page. In this case, semgrep detected a vulnerability called SQL 
``` bash              
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 1 (1 blocking)
 • Rules run: 74
 • Targets scanned: 1
 • Parsed lines: ~100.0%
 • No ignore information available
Ran 74 rules on 1 file: 1 finding.

A new version of Semgrep is available. See https://semgrep.dev/docs/upgrading
{
    "version": "1.159.0",
    "results": [
        {
            "check_id": "javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection",
            "path": "juice-shop/routes/login.ts",
            "start": {
                "line": 34,
                "col": 28,
                "offset": 1459
            },
            "end": {
                "line": 34,
                "col": 169,
                "offset": 1600
            },
            "extra": {
                "message": "Detected a sequelize statement that is tainted by user-input. This could lead to SQL injection if the variable is user-controlled and is not properly sanitized. In order to prevent SQL injection, it is recommended to use parameterized queries or prepared statements.",
                "metadata": {
                    "interfile": true,
                    "references": [
                        "https://sequelize.org/docs/v6/core-concepts/raw-queries/#replacements"
                    ],
                    "category": "security",
                    "technology": [
                        "express"
                    ],
                    "cwe": [
                        "CWE-89: Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection')"
                    ],
                    "owasp": [
                        "A01:2017 - Injection",
                        "A03:2021 - Injection",
                        "A05:2025 - Injection"
                    ],
                    "cwe2022-top25": true,
                    "cwe2021-top25": true,
                    "subcategory": [
                        "vuln"
                    ],
                    "likelihood": "HIGH",
                    "impact": "HIGH",
                    "confidence": "HIGH",
                    "license": "Semgrep Rules License v1.0. For more details, visit semgrep.dev/legal/rules-license",
                    "vulnerability_class": [
                        "SQL Injection"
                    ],
                    "source": "https://semgrep.dev/r/javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection",
                    "shortlink": "https://sg.run/gjoe"
                },
                "severity": "ERROR",
                "fingerprint": "requires login",
                "lines": "requires login",
                "validation_state": "NO_VALIDATOR",
                "engine_kind": "OSS"
            }
        }
    ],
    "errors": [],
    "paths": {
        "scanned": [
            "juice-shop/routes/login.ts"
        ]
    }
```
We can see that Semgrep detected this `"CWE-89: Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection')"`. So we went to the endpoint *juice-shop/routes/login.ts* to fix the line that was vulnerable to SQL Injection.

---
Remember that semgrep in its scan, it find the exact line where the vulnerability was. 
It was in the 34 line and says this:

```bash
models.sequelize.query(`SELECT * FROM Users WHERE email = '${req.body.email || ''}' AND password = '${security.hash(req.body.password || '')}' AND deletedAt IS NULL`, { model: UserModel, plain: true }) 
```
At first, *models.sequelize.query* is not the best tool to avoid SQL-Injection, because it calls the SQL engine directly and avoids going through Sequelize's safe methods.

---

To remediate the SQL Injection vulnerability, we replaced the `models.sequelize.query()` with `UserModel.findOne()` method. Unlike string concatenation, this method uses prepared statements — it sends the SQL instruction and the user input to the database separately, ensuring that any malicious input such as `' OR 1=1--` is treated as literal text and never interpreted as SQL code.
```bash
UserModel.findOne({
    where: {
    email: req.body.email || '',
    password: security.hash(req.body.password || ''),
    deletedAt: null
    }
})
```
After this, we use semgrep again in *juice-shop/routes/login.ts* to ensure that the vulnerability has been fixed. And we get this:
```bash
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 0 (0 blocking)
 • Rules run: 74
 • Targets scanned: 1
 • Parsed lines: ~100.0%
 • No ignore information available
Ran 74 rules on 1 file: 0 findings.
(need more rules? `semgrep login` for additional free Semgrep Registry rules)


A new version of Semgrep is available. See https://semgrep.dev/docs/upgrading
If Semgrep missed a finding, please send us feedback to let us know!
See https://semgrep.dev/docs/reporting-false-negatives/
{
    "version": "1.159.0",
    "results": [],
    "errors": [],
    "paths": {
        "scanned": [
            "juice-shop/routes/login.ts"
        ]
    }
}
```
Notice that it has been fixed, result's part appears empty which means that semgrep hasn't found a vulnerability in the login.ts file.