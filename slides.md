layout: true
class: middle

---

# The Template Method and Strategy Patterns

---

Its not hard to write software where the requirements never change

---

However, if you make something people like they always want more!

---

```ruby
class Report
  attr_reader :title, :items

  def initialize(title, items)
    @title = title
    @items = items
  end

  def generate
    lines = []

    lines << "<html>"
    lines << "  <head>"
    lines << "    <title>#{title}</title>"
    lines << "  </head>"
    lines << "  <body>"
    items.each do |item|
      lines << "    <p>#{item}</p>"
    end
    lines << "  </body>"
    lines << "</html>"

    lines.join("\n")
  end
end
```

---

```ruby
stuff_to_buy = ["Hot Pockets", "Frozen Pizza", "Pop Tarts"]
report = Report.new("Shopping List", stuff_to_buy)
puts(report.generate)
```

--

```bash
$ ruby report_generator.rb
<html>
  <head>
    <title>Shopping List</title>
  </head>
  <body>
    <p>Hot Pockets</p>
    <p>Frozen Pizza</p>
    <p>Pop Tarts</p>
  </body>
</html>
```

--

# Success!

---

#¯\\\_(ツ)\_/¯

- *Really* shouldn't generate HTML like this

--

- But it's simple and it works!

---

# How about some plain text!?

---

```ruby
class Report
  attr_reader :title, :items

  def initialize(title, items)
    @title = title
    @items = items
  end

  def generate(format)
    # ...
  end
end
```

---

```ruby
lines = []

case format
when :plain_text
  lines << title
  lines << "=" * title.length
when :html
  lines << "<html>"
  lines << "  <head>"
  lines << "    <title>#{title}</title>"
  lines << "  </head>"
  lines << "  <body>"
else
  raise ArgumentError
end

items.each do |item|
  case format
  when :plain_text
    lines << "- #{item}"
  when :html
    lines << "    <p>#{item}</p>"
  end
end

if format == :html
  lines << "  </body>"
  lines << "</html>"
end

lines.join("\n")
```

---

```ruby
stuff_to_buy = ["Hot Pockets", "Frozen Pizza", "Pop Tarts"]
report = Report.new("Shopping List", stuff_to_buy)
puts(report.generate(:plain_text))
```

--

```bash
$ ruby report_generator.rb
Shopping List
=============
- Hot Pockets
- Frozen Pizza
- Pop Tarts
```

--

# That was less fun

---

## Now we need LaTeX...

--

## And PostScript too!

--

![angry](http://i.giphy.com/xTiTnJ3BooiDs8dL7W.gif)

---


### Keep what's different seperate from what stays the same

--

1. Header
2. Title
3. Each item
4. Ending

---

```ruby
class Report
  attr_reader :title, :items

  def initialize(title, items)
    @title = title
    @items = items
  end

  def generate
    lines = []
    lines << generate_start
    lines << generate_head
    lines << generate_body_start
    lines << generate_body
    lines << generate_body_end
    lines << generate_end
    lines.flatten.compact
  end

  def generate_body
    items.map do |item|
      generate_item(item)
    end
  end
end
```

---

```ruby
class HTMLReport < Report
  def generate_start
    "<html>"
  end
  def generate_head
    [
      "  <head>",
      "    <title>#{title}</title>",
      "  </head>"
    ]
  end
  def generate_body_start
    "<body>"
  end
  def generate_item(item)
    "  <p>#{item}</p>"
  end
  def generate_body_end
    "</body>"
  end
  def generate_end
    "</html>"
  end
end
```

---

```ruby
class HTMLReport < Report
  def generate_start
    "<html>"
  end
  def generate_head
    [
      "  <head>",
      "    <title>#{title}</title>",
      "  </head>"
    ]
  end
  def generate_body_start
    "<body>"
  end
  def generate_item(item)
    "  <p>#{item}</p>"
  end
  def generate_body_end
    "</body>"
  end
  def generate_end
    "</html>"
  end
end
```

---

```ruby
class PlainTextReport < Report
  def generate_start
  end
  def generate_head
    [
      title,
      "=" * title.length
    ]
  end
  def generate_body_start
  end
  def generate_item(item)
    "- #{item}"
  end
  def generate_body_end
  end
  def generate_end
  end
end
```

---

```ruby
stuff_to_buy = ["Hot Pockets", "Frozen Pizza", "Pop Tarts"]

html_report = HTMLReport.new("Shopping List", stuff_to_buy)
puts(html_report.generate)

plain_text_report = PlainTextReport.new("Shopping List", stuff_to_buy)
puts(plain_text_report.generate)
```

---

### Different formats become different subclasses, the general flow stays in the base class

```
         +-------------------+
         |   AbstractClass   |
         +-------------------+
         |                   |
         | +templateMethod() |
         | +step1()          |
         | +step2()          |
         |                   |
         +-------------------+
                   #
                   |
        +----------+--------------+
        |                         |
+-------+-----------+   +---------+---------+
|   ConcreteClass1  |   |   ConcreteClass2  |
+-------------------+   +-------------------+
|                   |   |                   |
|     +step1()      |   |     +step1()      |
|     +step2()      |   |     +step2()      |
|                   |   |                   |
+-------------------+   +-------------------+

```

(Template Method's abstract class may also define hook methods that may be overridden by subclasses)

---

![Pitfall](https://upload.wikimedia.org/wikipedia/en/5/54/A2600_Pitfall.png)

# Pitfalls

- Inheritence increases coupling
- At its best when the interface imposed by the template method on the subclasses is minimal

---

# An alternative approach?

## Swap out the whole algorithm with the *Strategy Pattern*

![Swap](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2d/Set_of_security_screw_driver_bits.jpg/320px-Set_of_security_screw_driver_bits.jpg)

---

```ruby
class Report
  attr_reader :title, :items, :generator

  def initialize(title, items, generator)
    @title = title
    @items = items
    @generator = generator
  end

  def generate
    generator.generate
  end
end
```

---

```ruby
class HTMLGenerator
  def generate(title, items)
    lines = []
    lines << "<html>"
    lines << "  <head>"
    lines << "    <title>#{title}</title>"
    lines << "  </head>"
    lines << "  <body>"
    items.each do |item|
      lines << "    <p>#{item}</p>"
    end
    lines << "  </body>"
    lines << "</html>"

    lines.join("\n")
  end
end
```

---

```ruby
class PlainTextGenerator
  def generate(title, items)
    lines = []
    lines << title
    lines << "=" * title.length
    items.each do |item|
      lines << "- #{item}"
    end

    lines.join("\n")
  end
end
```

---

```ruby
stuff_to_buy = ["Hot Pockets", "Frozen Pizza", "Pop Tarts"]

html_report = Report.new("Shopping List", stuff_to_buy, HTMLGenerator.new)
puts(html_report.generate)

plain_text_report = Report.new("Shopping List", stuff_to_buy, PlainTextGenerator.new)
puts(plain_text_report.generate)
```

---

```
                               +---------------------+
+---------------------+        |      Strategy       |
|      Context        |        +---------------------+
+---------------------+        |   +doSomething()    |
|                     +------->|                     |
|                     |        +---------------------+
+---------------------+                  #
                                         |
                               +---------+-----------+
                               |                     |
                 +-------------+-------+   +---------+-----------+
                 |  ConcreteStrategyA  |   |  ConcreteStrategyB  |
                 +---------------------+   +---------------------+
                 |   +doSomething()    |   |   +doSomething()    |
                 |                     |   |                     |
                 +---------------------+   +---------------------+
```

---

[Design Patterns in Ruby](http://designpatternsinruby.com/)

