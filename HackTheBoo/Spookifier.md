# Spookifier

> There's a new trend of an application that generates a spooky name for you. Users of that application later discovered that their real names were also magically changed, causing havoc in their life. Could you help bring down this application?

First thing i did is try Server Side Template Injection, which allows RCE. Even if you are not sure from the source code whether it is vulnerable, you could try fuzzing in a few inputs. I tried `{{5*5}}`, `{5*5}` and `${5*5}` and found that `${5*5}` worked to display 25 on the webpage!

Perfect, now all we need to do is to read the flag with the payload `${open("/flag.txt").read()}`.

The Flag:

 ```
 HTB{t3mpl4t3_1nj3ct10n_1s_$p00ky!!}
 ```