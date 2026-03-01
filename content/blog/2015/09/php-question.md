---
title: "Interesting PHP question"
date: 2015-09-30
draft: false
---

Recently, I've come up with an interesting question that can potentially be used for interviewing candidates :)

We have such a PHP class.

```
class Collection implements Countable
{
    protected $items = [];
    
    public function __construct(array $data)
    {
        $this->items = $data;
    }
    
    public function count()
    {
        return count($this->items);
    }
}

$collection = new Collection(['one', 'two', 'three']);
```

Which is going to be faster: ```count($collection)``` or ```$collection->count()```? 
