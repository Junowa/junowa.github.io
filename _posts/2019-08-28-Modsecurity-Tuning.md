---
layout: post
author: junowa
---

This post is about modsecurity anomaly scoring.

Suite à une mise à jour de notre WAF (openresty/modsecurity), nous constatons des erreurs dans les logs modsec_audit.json.

```
{
    "transaction": {
        "client_ip": "62.23.81.210",
        "time_stamp": "Wed Aug 28 13:39:04 2019",
        "server_id": "032806661f3bf4b0de505bd5f6ea8671a8086b6f",
        "client_port": 29759,
        "host_ip": "62.23.81.210",
        "host_port": 443,
        "unique_id": "156699954488.321614",
        "request": {
            "method": "GET",
            "http_version": 1.1,
            "uri": "/groups/tdpportal/-/settings/ci_cd",
            "headers": {
                "Accept-Encoding": "gzip, deflate, br",
                "Cookie": "sidebar_collapsed=false; ai_user=xvYIS|2019-03-15T11:02:10.149Z; event_filter=all; collapsed_gutter=true; diff_view=parallel; frequently_used_emojis=thumbsup; sidebar_collapsed=false; SimpleSAMLSessionID=c9ce0aa7726978266d34c1e8672603b7; SimpleSAMLAuthToken=_0d26e13026eac2750446ecc530d91863783e4226fb; _gitlab_session=bcf421cc98eb308ea5ed3520e783bc5d",
                "Referer": "https://gitlab.thalesdigital.io/groups/tdpportal/-/edit",
                "Sec-Fetch-Site": "cross-site",
                "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36",
                "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
                "Cache-Control": "max-age=0",
                "Upgrade-Insecure-Requests": "1",
                "Sec-Fetch-Mode": "navigate",
                "Connection": "keep-alive",
                "Host": "gitlab.thalesdigital.io",
                "Accept-Language": "fr-FR,fr;q=0.9,en-US;q=0.8,en;q=0.7"
            }
        },
        "response": {
            "http_code": 200,
            "headers": {
                "X-Frame-Options": "DENY",
                "Etag": "W/\"0981aecaef59d0f78a0c05e8ff068560\"",
                "Cache-Control": "max-age=0, private, must-revalidate, no-store",
                "Strict-Transport-Security": "max-age=31536000",
                "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
                "Vary": "Accept-Encoding",
                "Pragma": "no-cache",
                "Connection": "keep-alive",
                "X-Content-Type-Options": "nosniff",
                "X-Runtime": "1.877574",
                "Content-Type": "text/html; charset=utf-8",
                "Date": "Wed, 28 Aug 2019 13:39:06 GMT",
                "Server": "nginx",
                "X-Request-Id": "bBu53f6RFT3",
                "X-Ua-Compatible": "IE=edge",
                "X-Xss-Protection": "1; mode=block",
                "Referrer-Policy": "strict-origin-when-cross-origin"
            }
        },
        "producer": {
            "modsecurity": "ModSecurity v3.0.3 (Linux)",
            "connector": "ModSecurity-nginx v1.0.0",
            "secrules_engine": "Enabled",
            "components": [
                "OWASP_CRS/3.0.0\""
            ]
        },
        "messages": [
            {
                "message": "postgres SQL Information Leakage",
                "details": {
                    "match": "Matched \"Operator `Rx' with parameter `(?i)(?:PostgreSQL query failed:|pg_query\\(\\) \\[:|pg_exec\\(\\) \\[:|PostgreSQL.*ERROR|Warning.*pg_.*|valid PostgreSQL result|Npgsql\\.|PG::([a-zA-Z]*)Error|Supplied argument is not a valid PostgreSQL (?:. (52 characters omitted)' against variable `RESPONSE_BODY' (Value: `<!DOCTYPE html>\\x0a<html class=\"\" lang=\"en\">\\x0a<head prefix=\"og: http://ogp.me/ns#\">\\x0a<meta chars (155468 characters omitted)' )",
                    "reference": "o136936,4105v981,148887",
                    "ruleId": "951240",
                    "file": "/usr/local/owasp-modsecurity-crs-3.0.0/rules/RESPONSE-951-DATA-LEAKAGES-SQL.conf",
                    "lineNumber": "421",
                    "data": "Matched Data:  found within TX:sql_error_match: 1",
                    "severity": "2",
                    "ver": "OWASP_CRS/3.0.0",
                    "rev": "1",
                    "tags": [],
                    "maturity": "7",
                    "accuracy": "9"
                }
            },
            {
                "message": "postgres SQL Information Leakage",
                "details": {
                    "match": "Matched \"Operator `Rx' with parameter `(?i)(?:PostgreSQL query failed:|pg_query\\(\\) \\[:|pg_exec\\(\\) \\[:|PostgreSQL.*ERROR|Warning.*pg_.*|valid PostgreSQL result|Npgsql\\.|PG::([a-zA-Z]*)Error|Supplied argument is not a valid PostgreSQL (?:. (52 characters omitted)' against variable `RESPONSE_BODY' (Value: `<!DOCTYPE html>\\x0a<html class=\"\" lang=\"en\">\\x0a<head prefix=\"og: http://ogp.me/ns#\">\\x0a<meta chars (155468 characters omitted)' )",
                    "reference": "o136936,4105v981,148887",
                    "ruleId": "951240",
                    "file": "/usr/local/owasp-modsecurity-crs-3.0.0/rules/RESPONSE-951-DATA-LEAKAGES-SQL.conf",
                    "lineNumber": "421",
                    "data": "Matched Data:  found within TX:sql_error_match: 1",
                    "severity": "2",
                    "ver": "OWASP_CRS/3.0.0",
                    "rev": "1",
                    "tags": [
                        "application-multi",
                        "language-multi",
                        "platform-pgsql",
                        "attack-disclosure",
                        "OWASP_CRS/LEAKAGE/ERRORS_SQL",
                        "CWE-209"
                    ],
                    "maturity": "7",
                    "accuracy": "9"
                }
            },
            {
                "message": "Outbound Anomaly Score Exceeded (Total Score: 5)",
                "details": {
                    "match": "Matched \"Operator `Ge' with parameter `5' against variable `TX:OUTBOUND_ANOMALY_SCORE' (Value: `5' )",
                    "reference": "",
                    "ruleId": "959100",
                    "file": "/usr/local/owasp-modsecurity-crs-3.0.0/rules/RESPONSE-959-BLOCKING-EVALUATION.conf",
                    "lineNumber": "26",
                    "data": "",
                    "severity": "0",
                    "ver": "",
                    "rev": "",
                    "tags": [
                        "anomaly-evaluation"
                    ],
                    "maturity": "0",
                    "accuracy": "0"
                }
            },
            {
                "message": "Outbound Anomaly Score Exceeded (score 5): postgres SQL Information Leakage",
                "details": {
                    "match": "Matched \"Operator `Ge' with parameter `5' against variable `TX:OUTBOUND_ANOMALY_SCORE' (Value: `5' )",
                    "reference": "",
                    "ruleId": "980140",
                    "file": "/usr/local/owasp-modsecurity-crs-3.0.0/rules/RESPONSE-980-CORRELATION.conf",
                    "lineNumber": "74",
                    "data": "",
                    "severity": "0",
                    "ver": "",
                    "rev": "",
                    "tags": [
                        "event-correlation"
                    ],
                    "maturity": "0",
                    "accuracy": "0"
                }
            }
        ]
    }
```

On s'aperçoit ici que la règle déclenchée est la **980140** incluses dans **RESPONSE-980-CORRELATION.conf**.

Cette règle fonctionne par corrélation au niveau de réponses des serveurs. La corrélation modsecurity s'appuit sur le mode **anomaly scoring**. C'est le mode par défaut depuis crs-3.0.

https://www.modsecurity.org/CRS/Documentation/anomaly.html

La configuration **anomaly scoring** s'effectue dans crs-setup.conf.

Chaque règle se voit affecter plusieurs scores (global, par famille d'attaques).

Lorsque la règle matche, les scores sont incrémentés en fonction du **severity level** prédéfini.
Les niveaus sont : critical, error, warning, notice et peuvent être modifiés dans crs-setup.conf.

La régle de corrélation applique ensuite une action (blocage dans notre cas) lorsqu'un seuil est atteint.

Les seuils sont configurés aussi dans csr-setup.conf:

````
SecAction \
 "id:'900003',\
  phase:1,\
  nolog,\
  pass,\
  t:none,\
  setvar:tx.sql_injection_score_threshold=15,\
  setvar:tx.xss_score_threshold=15,\
  setvar:tx.rfi_score_threshold=5,\
  setvar:tx.lfi_score_threshold=5,\
  setvar:tx.rce_score_threshold=5,\
  setvar:tx.command_injection_score_threshold=5,\
  setvar:tx.php_injection_score_threshold=5,\
  setvar:tx.http_violation_score_threshold=5,\
  setvar:tx.trojan_score_threshold=5,\
  setvar:tx.session_fixation_score_threshold=5,\
  setvar:tx.inbound_anomaly_score_threshold=5,\
  setvar:tx.outbound_anomaly_score_threshold=4"
  ````

En modifiant les seuils, il est possible de rendre l'anomaly scoring légèrement plus permissif pour gérer les cas de faux positif.

En effet, dans notre cas, la règle dont le score s'incrémente, concerne un faux positif de type postgresql leakage dans la réponse. Le body de la réponse match la regex **PostgreSQL.*ERROR** car les chaînes postgesql et error sont présentes de manière complétement licite.

