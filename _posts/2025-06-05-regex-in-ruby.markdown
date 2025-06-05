---
layout: post
title:  "Regex in Ruby"
date:   2025-06-05 11:00:00 +0545
categories: [Ruby, regex]
tags: [ruby, regex]
---

# Ruby Regex: Pattern Matching Made Simple

Regular expressions (regex) in Ruby are powerful tools for pattern matching and text manipulation. Let's dive into the essentials with practical examples you can use immediately.

## Basic Syntax

Ruby provides two ways to create regex patterns:

```ruby
# Literal notation
pattern = /hello/

# Constructor method
pattern = Regexp.new("hello")
```

## Common Matching Methods

### `match` - Returns MatchData object
```ruby
email = "user@example.com"
result = email.match(/@(.+)\.(.+)/)
puts result[1]  # "example"
puts result[2]  # "com"
```

### `=~` - Returns position of match
```ruby
text = "The price is $25"
position = text =~ /\$\d+/
puts position  # 13
```

### `scan` - Returns all matches
```ruby
text = "Call me at 123-456-7890 or 987-654-3210"
phones = text.scan(/\d{3}-\d{3}-\d{4}/)
puts phones  # ["123-456-7890", "987-654-3210"]
```

## Essential Patterns

| Pattern | Meaning | Example |
|---------|---------|---------|
| `\d` | Digit | `/\d{3}/` matches "123" |
| `\w` | Word character | `/\w+/` matches "hello" |
| `\s` | Whitespace | `/\s+/` matches spaces |
| `.` | Any character | `/h.llo/` matches "hello" |
| `^` | Start of string | `/^Hello/` |
| `$` | End of string | `/world$/` |

## Practical Examples

### 1. Email Validation
```ruby
def valid_email?(email)
  email.match?(/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i)
end

puts valid_email?("test@example.com")  # true
puts valid_email?("invalid-email")     # false
```

### 2. Phone Number Extraction
```ruby
text = "Contact: (555) 123-4567 or 555.987.6543"
phones = text.scan(/\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}/)
puts phones  # ["(555) 123-4567", "555.987.6543"]
```

### 3. URL Parser
```ruby
url = "https://www.example.com/path?param=value"
match = url.match(/^(https?):\/\/([^\/]+)(\/[^?]*)?\??(.*)$/)

puts "Protocol: #{match[1]}"  # https
puts "Domain: #{match[2]}"    # www.example.com
puts "Path: #{match[3]}"      # /path
puts "Query: #{match[4]}"     # param=value
```

### 4. Password Strength Checker
```ruby
def strong_password?(password)
  return false if password.length < 8
  return false unless password.match?(/[A-Z]/)     # uppercase
  return false unless password.match?(/[a-z]/)     # lowercase
  return false unless password.match?(/\d/)        # digit
  return false unless password.match?(/[!@#$%^&*]/) # special char
  true
end

puts strong_password?("MyPass123!")  # true
puts strong_password?("weak")        # false
```

### 5. Text Cleaning and Replacement
```ruby
# Remove extra whitespace
text = "Too    many     spaces"
clean = text.gsub(/\s+/, " ")
puts clean  # "Too many spaces"

# Extract hashtags
tweet = "Loving #ruby and #programming today!"
hashtags = tweet.scan(/#\w+/)
puts hashtags  # ["#ruby", "#programming"]

# Mask credit card numbers
card = "My card number is 1234-5678-9012-3456"
masked = card.gsub(/\d{4}-\d{4}-\d{4}-(\d{4})/, "****-****-****-\\1")
puts masked  # "My card number is ****-****-****-3456"
```

## Modifiers

Add flags after the closing `/` to modify behavior:

```ruby
# Case insensitive
/hello/i.match("HELLO")  # matches

# Multiline mode
/^start/m.match("line1\nstart here")  # matches

# Extended mode (ignore whitespace)
pattern = /
  \d{3}    # area code
  -        # separator
  \d{4}    # number
/x
```

## Quick Tips

1. **Use `match?` for boolean checks** - it's faster than `match` when you only need true/false
2. **Escape special characters** with backslash: `\.`, `\$`, `\(`
3. **Use raw strings** for complex patterns: `%r{pattern}` instead of `/pattern/`
4. **Test your regex** - use online tools or IRB to verify patterns

## Common Gotchas

```ruby
# Wrong: . matches any character
"hello world".match(/hello.world/)  # matches "hello world"

# Right: escape the dot
"hello.world".match(/hello\.world/)  # matches "hello.world"

# Wrong: greedy matching
"<tag>content</tag>".match(/<.+>/)  # matches entire string

# Right: non-greedy matching
"<tag>content</tag>".match(/<.+?>/)  # matches "<tag>"
```

Ruby's regex engine is both powerful and intuitive. Start with these examples and gradually build more complex patterns as needed. Remember: readable code is better than clever regex - use them wisely!

{% include inarticle-adsense.html %}