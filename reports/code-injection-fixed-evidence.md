# Evidence SQL Injection fixed
Running semgrep on *juice-shop/routes/userProfile.ts*
We notice that there's a problem that could affect all the web page. In this case, semgrep detected a vulnerability called Eval Injection (Code injection)

``` bash
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 1 file tracked by git with 74 Code rules:
  Scanning 1 file with 74 ts rules.
  ---------------------------------------- 100% 0:00:00                                                                                                                        
                
                
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
            "check_id": "javascript.lang.security.audit.code-string-concat.code-string-concat",
            "path": "juice-shop\\routes\\userProfile.ts",
            "start": {
                "line": 62,
                "col": 20,
                "offset": 1916
            },
            "end": {
                "line": 62,
                "col": 30,
                "offset": 1926
            },
            "extra": {
                "message": "Found data from an Express or Next web request flowing to `eval`. If this data is user-controllable this can lead to execution of arbitrary system commands in the context of your application process. Avoid `eval` whenever possible.",
                "metadata": {
                    "interfile": true,
                    "confidence": "HIGH",
                    "owasp": [
                        "A03:2021 - Injection",
                        "A05:2025 - Injection"
                    ],
                    "cwe": [
                        "CWE-95: Improper Neutralization of Directives in Dynamically Evaluated Code ('Eval Injection')"
                    ],
                    "references": [
                        "https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval",
                        "https://nodejs.org/api/child_process.html#child_processexeccommand-options-callback",
                        "https://www.stackhawk.com/blog/nodejs-command-injection-examples-and-prevention/",
                        "https://ckarande.gitbooks.io/owasp-nodegoat-tutorial/content/tutorial/a1_-_server_side_js_injection.html"
                    ],
                    "category": "security",
                    "technology": [
                        "node.js",
                        "Express",
                        "Next.js"
                    ],
                    "subcategory": [
                        "vuln"
                    ],
                    "likelihood": "MEDIUM",
                    "impact": "MEDIUM",
                    "license": "Semgrep Rules License v1.0. For more details, visit semgrep.dev/legal/rules-license",
                    "vulnerability_class": [
                        "Code Injection"
                    ],
                    "source": "https://semgrep.dev/r/javascript.lang.security.audit.code-string-concat.code-string-concat",
                    "shortlink": "https://sg.run/96Yk"
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
            "juice-shop\\routes\\userProfile.ts"
        ]
    }
}
```
We can see that Semgrep detected this `"CWE-95: Improper Neutralization of Directives in Dynamically Evaluated Code ('Eval Injection')"`. So we went to the endpoint *juice-shop/routes/userProfile.ts* to fix the line that was vulnerable to Eval Injection.

---
Remember that semgrep in its scan, it find the exact line where the vulnerability was. 
It was in the 62 line and says this:

``` bash
 username = eval(code) // eslint-disable-line no-eval
```
Virtually any problem you try to solve with eval() can be safely solved using native data structures (such as objects or maps) or JSON.parse()

---

To fix this issue while maintaining the flow structure in case the application expects the text inside the brackets #{} to be processed, we must treat the content as plain text and never pass it through a code evaluator.

``` bash
username = String(code)
```

After this, we use semgrep again in juice-shop/routes/login.ts to ensure that the vulnerability has been fixed. And we get this:

``` bash
(.venv) PS D:\repositorios\ssslc-proyect-OWASP-juice-shop> semgrep --config=p/javascript juice-shop/routes/userProfile.ts --json | python -m json.tool > reports/code_injection_reporte.json                 
               
               
┌─────────────┐
│ Scan Status │
└─────────────┘
  Scanning 1 file tracked by git with 74 Code rules:
  Scanning 1 file with 74 ts rules.
  ---------------------------------------- 100% 0:00:00                                                                                                                        
                
                
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
            "juice-shop\\routes\\userProfile.ts"
        ]
    }
}
```