---
layout: post
title:  "Ruby Enumerable group_by"
date:   2025-06-04 22:25:58 +0545
categories: [Ruby, multithreading]
tags: [ruby, multithreading]
---

# Ruby Multithreading: A Practical Guide for Better Performance

Ruby's threading capabilities allow developers to write concurrent programs that can improve performance and responsiveness. While Ruby's Global Interpreter Lock (GIL) presents some limitations, understanding multithreading is crucial for building efficient applications, especially for I/O-bound operations.

## Understanding Ruby Threads

Ruby threads are lightweight units of execution that run concurrently within a single process. They're particularly effective for I/O-bound tasks like file operations, network requests, and database queries, where threads can work while others wait for external resources.

### Basic Thread Creation

```ruby
# Simple thread creation
thread = Thread.new do
  puts "Hello from thread #{Thread.current.object_id}"
  sleep(2)
  puts "Thread finished"
end

puts "Main thread continues..."
thread.join  # Wait for thread to complete
puts "All done!"
```

## Practical Example 1: Concurrent Web Scraping

One of the most common use cases for threading is making multiple HTTP requests simultaneously:

```ruby
require 'net/http'
require 'uri'
require 'json'

class ConcurrentWebScraper
  def initialize(urls)
    @urls = urls
    @results = {}
    @mutex = Mutex.new
  end

  def scrape_sequentially
    start_time = Time.now
    
    @urls.each do |url|
      response = fetch_url(url)
      @results[url] = response
    end
    
    puts "Sequential execution: #{Time.now - start_time} seconds"
    @results
  end

  def scrape_concurrently
    start_time = Time.now
    threads = []
    
    @urls.each do |url|
      threads << Thread.new(url) do |current_url|
        response = fetch_url(current_url)
        
        # Thread-safe access to shared data
        @mutex.synchronize do
          @results[current_url] = response
        end
      end
    end
    
    # Wait for all threads to complete
    threads.each(&:join)
    
    puts "Concurrent execution: #{Time.now - start_time} seconds"
    @results
  end

  private

  def fetch_url(url)
    uri = URI(url)
    response = Net::HTTP.get_response(uri)
    {
      status: response.code,
      length: response.body.length,
      title: extract_title(response.body)
    }
  rescue => e
    { error: e.message }
  end

  def extract_title(html)
    title_match = html.match(/<title>(.*?)<\/title>/i)
    title_match ? title_match[1] : "No title found"
  end
end

# Usage example
urls = [
  'https://www.ruby-lang.org',
  'https://github.com',
  'https://stackoverflow.com',
  'https://www.google.com'
]

scraper = ConcurrentWebScraper.new(urls)

puts "=== Sequential Scraping ==="
sequential_results = scraper.scrape_sequentially

puts "\n=== Concurrent Scraping ==="
concurrent_results = scraper.scrape_concurrently

puts "\nResults:"
concurrent_results.each do |url, data|
  puts "#{url}: #{data[:status]} - #{data[:title]}"
end
```

## Practical Example 2: File Processing with Thread Pool

For CPU-intensive tasks or when you need to control the number of concurrent threads:

```ruby
class ThreadPool
  def initialize(size = 4)
    @size = size
    @jobs = Queue.new
    @pool = Array.new(@size) do
      Thread.new do
        catch(:exit) do
          loop do
            job, args, block = @jobs.pop
            case job
            when :work
              block.call(*args)
            when :exit
              throw :exit
            end
          end
        end
      end
    end
  end

  def schedule(*args, &block)
    @jobs << [:work, args, block]
  end

  def shutdown
    @size.times do
      @jobs << [:exit]
    end
    @pool.each(&:join)
  end
end

class FileProcessor
  def initialize(thread_count = 4)
    @thread_pool = ThreadPool.new(thread_count)
    @results = []
    @mutex = Mutex.new
  end

  def process_files(file_paths)
    file_paths.each do |file_path|
      @thread_pool.schedule(file_path) do |path|
        result = process_single_file(path)
        
        @mutex.synchronize do
          @results << result
        end
      end
    end
    
    # Wait a bit for processing to complete
    sleep(1) while @results.length < file_paths.length
    
    @thread_pool.shutdown
    @results
  end

  private

  def process_single_file(file_path)
    return { file: file_path, error: "File not found" } unless File.exist?(file_path)
    
    content = File.read(file_path)
    
    {
      file: file_path,
      size: content.length,
      lines: content.lines.count,
      words: content.split.count,
      processed_at: Time.now,
      thread_id: Thread.current.object_id
    }
  rescue => e
    { file: file_path, error: e.message }
  end
end

# Create sample files for demonstration
sample_files = []
5.times do |i|
  filename = "sample_#{i}.txt"
  File.write(filename, "Sample content for file #{i}\n" * (i + 1) * 10)
  sample_files << filename
end

# Process files concurrently
processor = FileProcessor.new(3)
results = processor.process_files(sample_files)

puts "File Processing Results:"
results.each do |result|
  if result[:error]
    puts "Error processing #{result[:file]}: #{result[:error]}"
  else
    puts "#{result[:file]}: #{result[:lines]} lines, #{result[:words]} words (Thread: #{result[:thread_id]})"
  end
end

# Cleanup
sample_files.each { |file| File.delete(file) if File.exist?(file) }
```

## Thread Synchronization and Safety

### Mutex for Thread Safety

```ruby
class ThreadSafeCounter
  def initialize
    @count = 0
    @mutex = Mutex.new
  end

  def increment
    @mutex.synchronize do
      @count += 1
    end
  end

  def decrement
    @mutex.synchronize do
      @count -= 1
    end
  end

  def value
    @mutex.synchronize do
      @count
    end
  end
end

# Demonstrate thread safety
counter = ThreadSafeCounter.new
threads = []

# Create 10 threads that increment counter 1000 times each
10.times do
  threads << Thread.new do
    1000.times { counter.increment }
  end
end

threads.each(&:join)
puts "Final counter value: #{counter.value}" # Should be 10,000
```

### Using Queue for Producer-Consumer Pattern

```ruby
class ProducerConsumer
  def initialize
    @queue = Queue.new
    @results = []
    @mutex = Mutex.new
  end

  def start_processing(items_to_process)
    # Producer thread
    producer = Thread.new do
      items_to_process.each do |item|
        @queue << item
        puts "Produced: #{item}"
        sleep(0.1) # Simulate work
      end
      
      # Signal completion
      3.times { @queue << :done }
    end

    # Consumer threads
    consumers = 3.times.map do |i|
      Thread.new do
        loop do
          item = @queue.pop
          break if item == :done
          
          # Simulate processing
          processed = process_item(item)
          
          @mutex.synchronize do
            @results << processed
            puts "Consumer #{i} processed: #{processed}"
          end
          
          sleep(0.2) # Simulate work
        end
      end
    end

    # Wait for completion
    producer.join
    consumers.each(&:join)
    
    @results
  end

  private

  def process_item(item)
    "processed_#{item}_#{Time.now.to_f}"
  end
end

# Usage
pc = ProducerConsumer.new
items = (1..10).to_a
results = pc.start_processing(items)

puts "\nFinal results:"
results.each { |result| puts result }
```

## Practical Example 3: Concurrent Database Operations

```ruby
require 'sqlite3'

class ConcurrentDatabase
  def initialize(db_path = 'concurrent_example.db')
    @db_path = db_path
    setup_database
  end

  def setup_database
    db = SQLite3::Database.new(@db_path)
    db.execute <<-SQL
      CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY,
        name TEXT,
        email TEXT,
        created_at DATETIME,
        thread_id TEXT
      )
    SQL
    db.close
  end

  def insert_users_sequentially(users)
    start_time = Time.now
    
    db = SQLite3::Database.new(@db_path)
    users.each do |user|
      db.execute(
        "INSERT INTO users (name, email, created_at, thread_id) VALUES (?, ?, ?, ?)",
        [user[:name], user[:email], Time.now, 'main']
      )
    end
    db.close
    
    puts "Sequential inserts: #{Time.now - start_time} seconds"
  end

  def insert_users_concurrently(users)
    start_time = Time.now
    threads = []
    
    users.each_slice(users.length / 4) do |user_batch|
      threads << Thread.new do
        db = SQLite3::Database.new(@db_path)
        user_batch.each do |user|
          db.execute(
            "INSERT INTO users (name, email, created_at, thread_id) VALUES (?, ?, ?, ?)",
            [user[:name], user[:email], Time.now, Thread.current.object_id.to_s]
          )
        end
        db.close
      end
    end
    
    threads.each(&:join)
    puts "Concurrent inserts: #{Time.now - start_time} seconds"
  end

  def get_stats
    db = SQLite3::Database.new(@db_path)
    total = db.execute("SELECT COUNT(*) FROM users")[0][0]
    by_thread = db.execute("SELECT thread_id, COUNT(*) FROM users GROUP BY thread_id")
    db.close
    
    puts "Total users: #{total}"
    puts "By thread:"
    by_thread.each do |thread_id, count|
      puts "  Thread #{thread_id}: #{count} users"
    end
  end

  def cleanup
    File.delete(@db_path) if File.exist?(@db_path)
  end
end

# Generate sample data
users = 1000.times.map do |i|
  {
    name: "User #{i}",
    email: "user#{i}@example.com"
  }
end

# Test concurrent database operations
db = ConcurrentDatabase.new

puts "=== Sequential Database Operations ==="
db.insert_users_sequentially(users[0...250])

puts "\n=== Concurrent Database Operations ==="
db.insert_users_concurrently(users[250...1000])

puts "\n=== Database Statistics ==="
db.get_stats

# Cleanup
db.cleanup
```

## Best Practices and Common Pitfalls

### 1. Always Join Your Threads

```ruby
# Bad: Thread may not complete
Thread.new { expensive_operation }

# Good: Ensure thread completion
thread = Thread.new { expensive_operation }
thread.join
```

### 2. Handle Exceptions Properly

```ruby
thread = Thread.new do
  begin
    risky_operation
  rescue => e
    puts "Thread error: #{e.message}"
  end
end

thread.join

# Check for exceptions
if thread.status.nil?
  puts "Thread completed"
elsif thread.status == false
  puts "Thread terminated with exception"
end
```

### 3. Avoid Race Conditions

```ruby
# Bad: Race condition
@shared_data = []
threads = 5.times.map do
  Thread.new do
    @shared_data << "data"  # Unsafe
  end
end

# Good: Thread-safe access
@shared_data = []
@mutex = Mutex.new

threads = 5.times.map do
  Thread.new do
    @mutex.synchronize do
      @shared_data << "data"  # Safe
    end
  end
end
```

## When to Use Ruby Threading

### Ideal Use Cases:
- **I/O-bound operations**: File reading, network requests, database queries
- **Producer-consumer scenarios**: Background job processing
- **Parallel data processing**: When tasks can be divided into independent chunks
- **User interface responsiveness**: Keeping UI responsive during long operations

### When NOT to Use:
- **CPU-intensive tasks**: Ruby's GIL limits true parallelism
- **Simple sequential operations**: Threading overhead may not be worth it
- **Shared mutable state**: Without proper synchronization, leads to bugs

## Performance Considerations

Ruby's Global Interpreter Lock (GIL) means that only one thread can execute Ruby code at a time. However, threads are still valuable because:

1. **I/O operations release the GIL**: Threads can run truly concurrently during I/O
2. **Better resource utilization**: While one thread waits, others can work
3. **Improved responsiveness**: Applications feel more responsive to users

## Key Takeaways

- Ruby threading excels at I/O-bound tasks despite the GIL limitation
- Always use proper synchronization (Mutex, Queue) for shared data
- Thread pools help manage resource usage and prevent thread explosion
- Handle exceptions within threads to prevent silent failures
- Consider alternatives like processes or async libraries for CPU-intensive work

{% include inarticle-adsense.html %}