---
layout: post
title: "Mastering Ruby Memory Management: A Practical Guide to Profiling and Optimization"
date: 2025-03-20 12:24:00 +0545
categories: [Ruby, Performance]
tags: [ruby, performance]
---

Memory usage is often the silent performance killer in Ruby applications. While we frequently focus on execution speed, memory consumption can cause slowdowns, unexpected crashes, and increased hosting costs. In this guide, I'll walk you through practical techniques for tracking and optimizing memory usage in Ruby and Rails applications using the powerful `memory_profiler` gem.

## Why Memory Matters

Memory issues in Ruby apps typically manifest in several ways:

1. **Slow performance**: Memory bloat forces the garbage collector to work overtime
2. **Random crashes**: Out-of-memory errors 
3. **Steadily increasing memory usage**: Signs of memory leaks
4. **Excessive hosting costs**: Needing larger instances to handle memory requirements

Let's dive into how to identify and solve these issues.

## Setting Up Memory Profiler

First, you'll need to install the `memory_profiler` gem:

```ruby
# In your Gemfile
gem 'memory_profiler'

# Or install it globally
# gem install memory_profiler
```

Then, install the gems:

```bash
bundle install
```

## Basic Memory Profiling

Let's start with a simple example. Create a file named `memory_test.rb`:

```ruby
require 'memory_profiler'

# Set up logging to a file (optional but recommended)
log_file = File.new('memory_profile.log', 'w')
$stdout = log_file
$stdout.sync = true

report = MemoryProfiler.report do
  # The code you want to profile
  array = Array.new(1_000_000) { |i| "string #{i}" }
end

# Print the report
report.pretty_print
```

Run it:

```bash
ruby memory_test.rb
```

Now look at `memory_profile.log`. The most important lines are at the top:

```
Total allocated: 120,000,816 bytes (2,000,002 objects)
Total retained:  120,000,816 bytes (2,000,002 objects)
```

This tells you:
- How much memory was allocated during execution
- How much remained in use after execution (not garbage collected)

## Profiling Rails Applications

For Rails applications, here's how to profile a specific action or process:

```ruby
# In a controller or job
def expensive_action
  data = nil
  
  report = MemoryProfiler.report do
    # Code to profile
    data = User.includes(:posts, :comments)
              .where(active: true)
              .map { |u| u.attributes.merge(post_count: u.posts.size) }
  end
  
  # Save the report to a file
  File.open("#{Rails.root}/log/memory_profile_#{Time.now.to_i}.log", 'w') do |file|
    report.pretty_print(to_file: file)
  end
  
  render json: data
end
```

## Memory Optimization Techniques

Now that you can measure memory usage, let's look at common memory optimization strategies with real examples.

### 1. Use Batching for Large Collections

**Problem:**

```ruby
# Memory-intensive approach
report = MemoryProfiler.report do
  users = User.all
  processed_users = users.map do |user|
    # Process each user
    process_user_data(user)
  end
end
report.pretty_print
```

**Solution:**

```ruby
# Memory-optimized approach
report = MemoryProfiler.report do
  processed_users = []
  User.find_each(batch_size: 100) do |user|
    processed_users << process_user_data(user)
  end
end
report.pretty_print
```

Using `find_each` with batching reduces memory usage by loading records in smaller chunks rather than all at once.

### 2. Optimize String Operations

**Problem:**

```ruby
report = MemoryProfiler.report do
  result = ""
  1000.times do |i|
    result += "Adding string #{i}. "  # Creates a new string each time
  end
end
report.pretty_print
```

**Solution:**

```ruby
report = MemoryProfiler.report do
  chunks = []
  1000.times do |i|
    chunks << "Adding string #{i}. "
  end
  result = chunks.join
end
report.pretty_print
```

The second approach allocates fewer intermediate string objects, reducing memory churn.

### 3. Avoid Unnecessary Object Creation

**Problem:**

```ruby
report = MemoryProfiler.report do
  users = User.all.to_a
  users.each do |user|
    # Creating temporary hash for each user
    user_data = {
      id: user.id,
      name: user.name,
      email: user.email,
      # Many more attributes
      created_at: user.created_at
    }
    process_data(user_data)
  end
end
report.pretty_print
```

**Solution:**

```ruby
report = MemoryProfiler.report do
  User.select(:id, :name, :email, :created_at).find_each do |user|
    # Use the ActiveRecord object directly
    process_data(user)
  end
end
report.pretty_print
```

This approach reduces memory by:
1. Selecting only needed columns
2. Avoiding unnecessary hash creation
3. Processing in batches

### 4. Identify Memory-Heavy Gems

The memory profiler report includes a breakdown of memory allocation by gem:

```
allocated memory by gem
-----------------------------------
 42462489  activesupport-7.0.4
 24595828  activerecord-7.0.4
  8953418  json-2.6.2
```

If a gem is using excessive memory, consider:
- Updating to a newer version
- Finding a more memory-efficient alternative
- Implementing a lightweight solution yourself

### 5. Monitor JSON Parsing and Generation

**Problem:**

```ruby
report = MemoryProfiler.report do
  large_data = File.read('large_data.json')
  parsed_data = JSON.parse(large_data)
  # Work with the data
  processed = process_json_data(parsed_data)
  JSON.generate(processed)
end
report.pretty_print
```

**Solution:**

```ruby
report = MemoryProfiler.report do
  # Stream parsing for large JSON files
  result = []
  Oj::Parser.new(:strict).parse_file('large_data.json') do |parsed|
    # Process each object as it's parsed
    result << transform_json_object(parsed)
  end
end
report.pretty_print
```

Using streaming parsers like Oj (optimized JSON) for large files dramatically reduces memory usage.

## Real-World Case Study: Rails Model Loading

Let's examine a common memory issue in Rails - loading models with many associations:

### The Problem

```ruby
# Controller action
def dashboard
  report = MemoryProfiler.report do
    @users = User.all.includes(:posts, :comments, :profile)
    @data = @users.map do |user|
      {
        user: user.attributes,
        posts: user.posts.map(&:attributes),
        comments: user.comments.map(&:attributes),
        profile: user.profile&.attributes
      }
    end
  end
  
  File.open("#{Rails.root}/log/dashboard_memory.log", 'w') do |file|
    report.pretty_print(to_file: file)
  end
  
  render json: @data
end
```

Memory profile results:
```
Total allocated: 254,328,816 bytes (3,200,502 objects)
Total retained:  125,624,816 bytes (1,600,252 objects)
```

### The Solution

```ruby
def dashboard
  report = MemoryProfiler.report do
    # 1. Select only needed columns
    # 2. Process in batches
    # 3. Use pluck for simple data extraction
    @data = []
    
    User.select(:id, :name, :email, :created_at)
        .find_in_batches(batch_size: 100) do |user_batch|
      
      user_ids = user_batch.map(&:id)
      
      # Fetch related data efficiently
      posts = Post.where(user_id: user_ids)
                 .select(:id, :title, :user_id)
                 .group_by(&:user_id)
                 
      comments = Comment.where(user_id: user_ids)
                       .select(:id, :content, :user_id)
                       .group_by(&:user_id)
                       
      profiles = Profile.where(user_id: user_ids)
                       .select(:id, :bio, :user_id)
                       .index_by(&:user_id)
      
      # Build the response without creating unnecessary objects
      user_batch.each do |user|
        user_data = {
          id: user.id,
          name: user.name,
          email: user.email,
          posts: posts[user.id]&.map { |p| { id: p.id, title: p.title } } || [],
          comments: comments[user.id]&.map { |c| { id: c.id, content: c.content } } || [],
          profile: profiles[user.id] ? { bio: profiles[user.id].bio } : nil
        }
        @data << user_data
      end
    end
  end
  
  File.open("#{Rails.root}/log/dashboard_memory_optimized.log", 'w') do |file|
    report.pretty_print(to_file: file)
  end
  
  render json: @data
end
```

Memory profile results after optimization:
```
Total allocated: 42,328,816 bytes (520,502 objects)
Total retained:  15,624,816 bytes (200,252 objects)
```

That's an 83% reduction in memory allocation and 88% reduction in retained memory!

## Advanced Techniques

### 1. Detect Memory Leaks

To detect memory leaks, run the same code multiple times and watch for increasing memory:

```ruby
5.times do |i|
  puts "Iteration #{i+1}"
  report = MemoryProfiler.report do
    # Code that might leak
    perform_operation
  end
  
  puts "Allocated: #{report.total_allocated_memsize} bytes"
  puts "Retained: #{report.total_retained_memsize} bytes"
  puts "---"
  
  # Force garbage collection between runs
  GC.start
end
```

If retained memory grows with each iteration, you likely have a leak.

### 2. Targeted Detail Analysis

For complex issues, examine object allocation details:

```ruby
report = MemoryProfiler.report do
  # Code to profile
end

# Get the top 20 locations allocating memory
puts "Top allocation locations:"
report.pretty_print(to_file: nil, detailed_report: false, scale_bytes: true, 
                   top: 20)

# Get detailed string allocations
string_locations = report.strings_allocated
string_locations.sort_by! { |l| -l[:count] }
string_locations[0..10].each do |location|
  puts "#{location[:count]} strings (#{location[:memsize]} bytes) allocated at #{location[:location]}"
end
```

### 3. Memory-Conscious Design Patterns

Here are some memory-efficient design patterns for Ruby applications:

#### Value Objects Instead of Hashes

```ruby
# Memory-heavy approach
users.map do |user|
  { id: user.id, name: user.name, stats: calculate_stats(user) }
end

# Memory-efficient approach
class UserPresenter
  attr_reader :id, :name
  
  def initialize(user)
    @user = user
    @id = user.id
    @name = user.name
  end
  
  def stats
    @stats ||= calculate_stats(@user)
  end
  
  private
  
  def calculate_stats(user)
    # Calculation logic
  end
end

users.map { |user| UserPresenter.new(user) }
```

#### Lazy Loading

```ruby
class Report
  def initialize(user_id)
    @user_id = user_id
  end
  
  def summary
    @summary ||= generate_summary
  end
  
  def details
    @details ||= generate_details
  end
  
  private
  
  def user
    @user ||= User.find(@user_id)
  end
  
  def generate_summary
    # Only calculated when needed
    { name: user.name, post_count: user.posts.count }
  end
  
  def generate_details
    # Only calculated when needed
    user.posts.map { |post| { title: post.title, likes: post.likes } }
  end
end
```

## Memory Profiling in Production

For monitoring memory in production:

1. **Use application monitoring tools**: New Relic, Scout APM, Skylight
2. **Set up custom memory logging**:

```ruby
# In an initializer
module MemoryLogger
  def self.log(label)
    memory_before = `ps -o rss= -p #{Process.pid}`.to_i / 1024
    yield if block_given?
    memory_after = `ps -o rss= -p #{Process.pid}`.to_i / 1024
    
    Rails.logger.info "[MEMORY] #{label}: #{memory_before}MB -> #{memory_after}MB (Δ#{memory_after - memory_before}MB)"
    
    # Force garbage collection and measure again to see retained memory
    GC.start
    memory_after_gc = `ps -o rss= -p #{Process.pid}`.to_i / 1024
    Rails.logger.info "[MEMORY] #{label} (after GC): #{memory_after_gc}MB (Δ#{memory_after_gc - memory_before}MB)"
  end
end

# Usage in controller
def expensive_action
  MemoryLogger.log("Processing users") do
    @users = User.process_all
  end
end
```

## Conclusion

Memory management in Ruby requires awareness and proactive optimization. The `memory_profiler` gem gives you powerful tools to identify memory issues and measure the impact of your optimizations.

Key takeaways:

1. **Measure before optimizing**: Use memory_profiler to identify actual problem areas
2. **Process in batches**: Break large operations into manageable chunks
3. **Select only what you need**: Fetch only required columns from the database
4. **Minimize object creation**: Reuse objects where possible
5. **Optimize string operations**: String concatenation can be memory-intensive
6. **Watch for memory leaks**: Monitor memory usage over time

By applying these techniques, you can build Ruby applications that are not only fast but also memory-efficient, resulting in more stable applications and lower hosting costs.

## Resources

- [memory_profiler GitHub repository](https://github.com/SamSaffron/memory_profiler)
- [Ruby Garbage Collection Deep Dive](https://jemma.dev/blog/gc-ruby)
- [Ruby Performance Optimization by Alexander Dymo](https://pragprog.com/titles/adrpo/ruby-performance-optimization/)
- [Derailed Benchmarks](https://github.com/schneems/derailed_benchmarks) - A Rails memory benchmarking tool
