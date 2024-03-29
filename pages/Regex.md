# [[Regex]] syntax
title:: Regex
	- A query string format for describing [[Regular expression]] . For examples, see [regexr.com](https://regexr.com/)]
		- Basic Regex Matching with alphabets
			- Let's say the word is `foobar200@huakuy.com`
			- `foo` -> `"foo"`
			- `foobar` -> `"foobar"`
			- `foolbar` -> `""`
			-
	- ## Structure
		- `/<regex>/[flag]>`
		- e.g. for example, `/foo/g` has string `foo` as its regex, and `g` is the (global) flag
	- ## Trailing flags
		- `g` - **g**lobal
		- `i` - case-**i**nsensitive
		- `m` - **m**ultiline
			- `/^The/m` will match every `The` that's the first word of its line.
		- `s` - **s**ingle line (_dotall_)
		- `u` - **u**nicode
		- `y` - stick**y**
	- ## Regex Matching
		- ### Regex Matching literal
			- Some chars we frequently use in strings are reserved as special characters, for example, `.` and `?`. If we want to actually match those characters, just escape - e.g. `/\./` and `/\?/`
			- `/\./` -> I love you**.** (matches the last full stop `.`)
			- `/.\./` -> I love yo**u.** (matches `u.`)
		- ### [[Regex special characters]]
			- > Note that only the body of the regex is shown, and no grouping is used in these Special Characters examples.
			- `[]` char set
				- `[f]at` -> Matt the **fat** cat
				- `[cf]at` -> Matt the **fat** **cat**
				- `[a-c]` -> Matt is a **bat** who loves fat **cat**s
			- `.` matches any char that's not `\n`
				- `.` -> **f****o****o****b****a****r** -> f + o + o + b + a + r
				- `.o` -> **fo**obar -> fo
				- `.bar` -> fo**obar** -> f + bar
			- `|` an OR operation
				- `t|The` -> **The** ba**t**man
			- `^` Start of line
				- `/^[t|T]he` -> **The** Batman is the best
				- `/[t|T]he` -> **The** Batman is **the** best
			- `$` EOL
			- `+` matches at least one of the preceding char
				- `e+` -> I don't **e**v**e**n know why s**ee**ds grow.
			- `?` _optionally_ matches the preceding char
				- `ou?` (`u` match is optional) -> My b**o**ss just b**ou**ght a new m**ou**se.
			- `*` matches 0 or more, like combining `+` with `?`
				- `es*` -> Edit th**e** Expr**ess**ion & T**e**xt to s**e****e** matches.
					- Note that with s**e****e**, the 2 `e` chars are matched separately by `e` (first item of the regex)
			- `\w` matches any **w**ordy chars (alphanumeric+underscore)
			- `\W` matches non-**w**ordy chars
			- `\s` matches any white**s**paces
			- `\S` matches any non-white**s**paces
			- `{i}` matches i chars of preceding token
				- `o{2}` -> f**oo**bar
				- `/w{5}` -> **RegEx**r was **creat**ed by **gskin**ner.com
				- `/w{7}` -> RegExr was **created** by **gskinne**r.com
				- `{i,}`
					- `\w{5,}` -> **RegExr** was **created** by **gskinner**.com
					- `\w{7,}` -> RegExr was **created** by **gskinner**.com
				- `{i,j}` matches between i to j chars of preceding token
					- `\w{5,6}` -> **RegExr** was **create**d by **gskinn**er.com
		- ### [[Regex grouping]]
			- Grouping is specified using parentheses `(regex)`
			- For example, we can use [[Regex grouping]] with the OR [[Regex special characters]] to match strings with more finesse:
				- `(t|T)he` -> **The** Batman is **the** world's best detective -> the + The
				- `(((d|D))o){1,2}` -> **Do**o dee **Do**nald **dodo**do dee **do** dee -> Do + Do + dodo + do
				- `(((d|D))o{2}){1,2}` -> **Doo** dee Donald doDodoDodoDo **doo****Doo** **doo** dee do dee -> Doo + dooDoo + doo
				- **Do**o dee **Do**nald **doDodoDodoDo** **do**o**Do**o **do**o dee **do** dee -> Do + Do + doDodoDodoDo + doDo + do do