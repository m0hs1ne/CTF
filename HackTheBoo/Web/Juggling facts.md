# Juggling Facts

> An organization seems to possess knowledge of the true nature of pumpkins. Can you find out what they honestly know and uncover this centuries-long secret once and for all?

## Solution

When i press `Secret Facts` it shows : `Secrets can only be accessed by admin`

[![Screen-Shot-2022-10-28-at-2-47-04-PM.png](https://i.postimg.cc/yYp2bxkP/Screen-Shot-2022-10-28-at-2-47-04-PM.png)](https://postimg.cc/VdM7SY9r)

Since this challenge’s name is Juggling Facts, I’ll google php juggling.

Now, we can dig deeper in this exploit: [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Type%20Juggling).

[![a5.png](https://i.postimg.cc/NFXwcrLm/a5.png)](https://postimg.cc/SYmPLR6x)

It seems like `IndexController.php` is vulnerable:
```PHP
 if ($jsondata['type'] === 'secrets' && $_SERVER['REMOTE_ADDR'] !== '127.0.0.1')
        {
            return $router->jsonify(['message' => 'Currently this type can be only accessed through localhost!']);
        }

        switch ($jsondata['type'])
        {
            case 'secrets':
                return $router->jsonify([
                    'facts' => $this->facts->get_facts('secrets')
                ]);
```

The first if statement is NOT vulnerable, as it’s using strict comparison (`===`, `!==`). So, we have to parse the type `POST` parameter.

However, the `switch` statement is vulnerable.

According to official [PHP documentation](https://www.php.net/manual/en/control-structures.switch.php) switch/case does [loose comparision](php.net/manual/en/types.comparisons.php#types.comparisions-loose).

Since 