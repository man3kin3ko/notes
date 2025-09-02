## Detection

From [Manual SQLi discovery tips](https://web.archive.org/web/20221128070017/https://gerbenjavado.com/manual-sql-injection-discovery-tips/):

You have to confirm it is the SQL command that is causing the error and not something like a date parser. To do this you can use a range of tricks:

- If a `'` is causing the error try to see if `\'` will result in success message (since the backslash cancels out the single quote in MySQL).
- You can also try if commenting out the `'` results in a success message like: `%23'` or `--'`. This is because you tell MySQL to explicitly ignore everything after the comment include the extra `'`.
- If a `'` is not allowed you can use comparisons between valid and invalid system variables like `@@versionz` vs `@@version` or invalid vs valid functions `SLEP(5)` vs `SLEEP(5)`.
- Sometimes your input will end up between `()` make sure you test `input)%23` as well to see if you can break out of these and exploit Union SQLi for example (`input) order by 5%23`).
- If the normal input is just an integer you can try to subtract some amount of it and see if that subtraction works (`id=460-5`).
- Try to see if an even amount of quotes results in a success message (for example `460''` or `460-''`) and an uneven amount results in an error (for example `460'` or `460-'''`).