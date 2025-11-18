---
layout: post
title: "Ruby Exception Handling"
date: 2025-06-05 11:00:00 +0545
categories: [Ruby, exception_handling]
tags: [ruby, exception_handling]
---

# Ruby Exception Handling: A Complete Guide to Robust Error Management

Exception handling is a critical aspect of writing robust Ruby applications. It allows you to gracefully handle errors, provide meaningful feedback to users, and prevent your application from crashing unexpectedly. Ruby provides a comprehensive exception handling system that gives developers fine-grained control over error management.

## Understanding Ruby Exceptions

In Ruby, exceptions are objects that represent errors or exceptional conditions. When an error occurs, Ruby "raises" an exception, which can be "caught" and handled appropriately. All exceptions in Ruby inherit from the `Exception` class.

### Ruby Exception Hierarchy

```ruby
Exception
 +-- NoMemoryError
 +-- ScriptError
 |    +-- LoadError
 |    +-- NotImplementedError
 |    +-- SyntaxError
 +-- SecurityError
 +-- SignalException
 |    +-- Interrupt
 +-- StandardError -- default for rescue
 |    +-- ArgumentError
 |    +-- IOError
 |    |    +-- EOFError
 |    +-- IndexError
 |    |    +-- KeyError
 |    |    +-- StopIteration
 |    +-- LocalJumpError
 |    +-- NameError
 |    |    +-- NoMethodError
 |    +-- RangeError
 |    |    +-- FloatDomainError
 |    +-- RegexpError
 |    +-- RuntimeError -- default for raise
 |    +-- SystemCallError
 |    |    +-- Errno::*
 |    +-- ThreadError
 |    +-- TypeError
 |    +-- ZeroDivisionError
 +-- SystemExit
 +-- SystemStackError
 +-- fatal -- impossible to rescue
```

## Basic Exception Handling with begin/rescue

The most common way to handle exceptions is using the `begin/rescue` block:

```ruby
begin
  # Code that might raise an exception
  risky_operation
rescue
  # Code to handle the exception
  puts "An error occurred!"
end

# Example with specific exception type
begin
  result = 10 / 0
rescue ZeroDivisionError
  puts "Cannot divide by zero!"
  result = nil
end
```

### Rescuing Multiple Exception Types

```ruby
begin
  # Risky code here
  perform_operation
rescue ZeroDivisionError
  puts "Division by zero error"
rescue ArgumentError
  puts "Invalid argument provided"
rescue StandardError => e
  puts "Other error occurred: #{e.message}"
end

# Alternative syntax for multiple exceptions
begin
  perform_operation
rescue ZeroDivisionError, ArgumentError => e
  puts "Math or argument error: #{e.message}"
rescue => e
  puts "Unexpected error: #{e.message}"
end
```

## Raising Exceptions

You can raise exceptions manually using the `raise` keyword:

```ruby
# Raise a generic RuntimeError
raise "Something went wrong!"

# Raise a specific exception type
raise ArgumentError, "Invalid input provided"

# Raise with custom exception class
raise ZeroDivisionError.new("Cannot divide by zero")

# Re-raise the current exception
begin
  1 / 0
rescue => e
  puts "Logging error: #{e.message}"
  raise  # Re-raises the same exception
end
```

### Creating Custom Exceptions

```ruby
# Define custom exception classes
class ValidationError < StandardError
  attr_reader :field, :value

  def initialize(field, value, message = nil)
    @field = field
    @value = value
    super(message || "Validation failed for #{field}: #{value}")
  end
end

class DatabaseConnectionError < StandardError
  def initialize(host, port)
    super("Failed to connect to database at #{host}:#{port}")
  end
end

# Usage
def validate_email(email)
  raise ValidationError.new(:email, email, "Invalid email format") unless email.include?("@")
end

begin
  validate_email("invalid-email")
rescue ValidationError => e
  puts "Field: #{e.field}, Value: #{e.value}"
  puts "Error: #{e.message}"
end
```

## Complete Exception Handling Syntax

Ruby provides several clauses for comprehensive exception handling:

```ruby
begin
  # Code that might raise an exception
  risky_operation
rescue SpecificError => e
  # Handle specific exceptions
  handle_specific_error(e)
rescue => e
  # Handle any StandardError
  handle_general_error(e)
else
  # Executed only if no exception was raised
  puts "Operation completed successfully"
ensure
  # Always executed, regardless of exceptions
  cleanup_resources
end
```

### The `ensure` Clause

The `ensure` clause is always executed, making it perfect for cleanup operations:

```ruby
def read_file(filename)
  file = nil
  begin
    file = File.open(filename, 'r')
    content = file.read
    return content
  rescue IOError => e
    puts "Error reading file: #{e.message}"
    return nil
  ensure
    # This always runs, even if an exception occurs
    file&.close
    puts "File handle closed"
  end
end
```

## Practical Example 1: File Processing with Exception Handling

```ruby
class FileProcessor
  class FileProcessingError < StandardError; end
  class InvalidFileTypeError < FileProcessingError; end
  class FileSizeError < FileProcessingError; end

  ALLOWED_EXTENSIONS = %w[.txt .csv .json].freeze
  MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB

  def initialize(log_errors: true)
    @log_errors = log_errors
    @processed_files = []
    @failed_files = []
  end

  def process_files(file_paths)
    file_paths.each do |path|
      begin
        process_single_file(path)
        @processed_files << path
        puts "✓ Successfully processed: #{path}"
      rescue FileProcessingError => e
        @failed_files << { path: path, error: e.message, type: e.class.name }
        log_error("Processing failed for #{path}: #{e.message}") if @log_errors
      rescue => e
        @failed_files << { path: path, error: e.message, type: "UnexpectedError" }
        log_error("Unexpected error processing #{path}: #{e.message}") if @log_errors
      end
    end

    generate_report
  end

  private

  def process_single_file(file_path)
    # Check if file exists
    raise FileProcessingError, "File not found: #{file_path}" unless File.exist?(file_path)

    # Validate file extension
    extension = File.extname(file_path).downcase
    unless ALLOWED_EXTENSIONS.include?(extension)
      raise InvalidFileTypeError, "Unsupported file type: #{extension}. Allowed: #{ALLOWED_EXTENSIONS.join(', ')}"
    end

    # Check file size
    file_size = File.size(file_path)
    if file_size > MAX_FILE_SIZE
      raise FileSizeError, "File too large: #{file_size} bytes (max: #{MAX_FILE_SIZE})"
    end

    # Process the file
    File.open(file_path, 'r') do |file|
      case extension
      when '.txt'
        process_text_file(file)
      when '.csv'
        process_csv_file(file)
      when '.json'
        process_json_file(file)
      end
    end
  end

  def process_text_file(file)
    content = file.read
    # Simulate processing
    raise FileProcessingError, "Text file is empty" if content.strip.empty?

    # Count lines and words
    lines = content.lines.count
    words = content.split.count
    puts "  Text file stats: #{lines} lines, #{words} words"
  end

  def process_csv_file(file)
    lines = file.readlines
    raise FileProcessingError, "CSV file has no data rows" if lines.length < 2

    puts "  CSV file stats: #{lines.length - 1} data rows"
  end

  def process_json_file(file)
    require 'json'
    content = file.read

    begin
      data = JSON.parse(content)
      puts "  JSON file stats: #{data.keys.count if data.is_a?(Hash)} top-level keys"
    rescue JSON::ParserError => e
      raise FileProcessingError, "Invalid JSON format: #{e.message}"
    end
  end

  def log_error(message)
    File.open('file_processing_errors.log', 'a') do |log_file|
      log_file.puts "[#{Time.now}] #{message}"
    end
  rescue => e
    puts "Failed to write to log file: #{e.message}"
  end

  def generate_report
    puts "\n" + "="*50
    puts "FILE PROCESSING REPORT"
    puts "="*50
    puts "Processed successfully: #{@processed_files.count}"
    puts "Failed to process: #{@failed_files.count}"

    unless @failed_files.empty?
      puts "\nFailed files:"
      @failed_files.each do |failure|
        puts "  ✗ #{failure[:path]}"
        puts "    Error: #{failure[:error]} (#{failure[:type]})"
      end
    end

    {
      successful: @processed_files,
      failed: @failed_files,
      total: @processed_files.count + @failed_files.count
    }
  end
end

# Create sample files for demonstration
sample_files = []

# Valid files
File.write('sample.txt', "This is a sample text file.\nWith multiple lines.")
File.write('data.csv', "name,age\nJohn,25\nJane,30")
File.write('config.json', '{"app": "demo", "version": "1.0"}')

# Invalid files
File.write('large_file.txt', "x" * (11 * 1024 * 1024))  # Too large
File.write('invalid.json', '{"invalid": json}')  # Invalid JSON
File.write('empty.txt', '')  # Empty file

sample_files = [
  'sample.txt', 'data.csv', 'config.json',
  'large_file.txt', 'invalid.json', 'empty.txt',
  'nonexistent.txt', 'document.pdf'  # Non-existent and unsupported type
]

# Process files with exception handling
processor = FileProcessor.new(log_errors: true)
report = processor.process_files(sample_files)

# Cleanup sample files
['sample.txt', 'data.csv', 'config.json', 'large_file.txt', 'invalid.json', 'empty.txt'].each do |file|
  File.delete(file) if File.exist?(file)
end
```

## Practical Example 2: Network Request Handler with Retry Logic

```ruby
require 'net/http'
require 'uri'
require 'json'

class NetworkRequestHandler
  class NetworkError < StandardError; end
  class TimeoutError < NetworkError; end
  class ServerError < NetworkError; end
  class ClientError < NetworkError; end

  def initialize(max_retries: 3, timeout: 10)
    @max_retries = max_retries
    @timeout = timeout
  end

  def fetch_with_retry(url, method: :get, payload: nil)
    attempt = 1

    begin
      puts "Attempt #{attempt}: Fetching #{url}"
      response = make_request(url, method, payload)

      case response.code.to_i
      when 200..299
        puts "✓ Success (#{response.code})"
        return parse_response(response)
      when 400..499
        raise ClientError, "Client error (#{response.code}): #{response.message}"
      when 500..599
        raise ServerError, "Server error (#{response.code}): #{response.message}"
      else
        raise NetworkError, "Unexpected response (#{response.code}): #{response.message}"
      end

    rescue Timeout::Error
      raise TimeoutError, "Request timed out after #{@timeout} seconds"
    rescue ServerError, TimeoutError => e
      # Retry on server errors and timeouts
      if attempt <= @max_retries
        wait_time = 2 ** (attempt - 1)  # Exponential backoff
        puts "  ✗ #{e.message}"
        puts "  Retrying in #{wait_time} seconds... (#{attempt}/#{@max_retries})"
        sleep(wait_time)
        attempt += 1
        retry
      else
        puts "  ✗ Max retries exceeded"
        raise e
      end
    rescue ClientError, NetworkError => e
      # Don't retry on client errors
      puts "  ✗ #{e.message}"
      raise e
    rescue => e
      # Handle unexpected errors
      puts "  ✗ Unexpected error: #{e.message}"
      raise NetworkError, "Unexpected error: #{e.message}"
    end
  end

  def fetch_multiple(urls)
    results = {}

    urls.each do |url|
      begin
        results[url] = fetch_with_retry(url)
      rescue NetworkError => e
        results[url] = { error: e.message, type: e.class.name }
      rescue => e
        results[url] = { error: e.message, type: "UnexpectedError" }
      end
    end

    generate_summary(results)
  end

  private

  def make_request(url, method, payload)
    uri = URI(url)

    Net::HTTP.start(uri.host, uri.port,
                   use_ssl: uri.scheme == 'https',
                   open_timeout: @timeout,
                   read_timeout: @timeout) do |http|

      case method
      when :get
        http.get(uri.path.empty? ? '/' : uri.path)
      when :post
        request = Net::HTTP::Post.new(uri.path)
        request.body = payload.to_json if payload
        request['Content-Type'] = 'application/json'
        http.request(request)
      else
        raise ArgumentError, "Unsupported HTTP method: #{method}"
      end
    end
  end

  def parse_response(response)
    content_type = response['content-type'] || ''

    if content_type.include?('application/json')
      begin
        JSON.parse(response.body)
      rescue JSON::ParserError => e
        raise NetworkError, "Invalid JSON response: #{e.message}"
      end
    else
      {
        content_type: content_type,
        body_length: response.body.length,
        headers: response.to_hash
      }
    end
  end

  def generate_summary(results)
    successful = results.count { |_, result| !result.key?(:error) }
    failed = results.count { |_, result| result.key?(:error) }

    puts "\n" + "="*50
    puts "NETWORK REQUEST SUMMARY"
    puts "="*50
    puts "Total requests: #{results.count}"
    puts "Successful: #{successful}"
    puts "Failed: #{failed}"

    unless failed.zero?
      puts "\nFailed requests:"
      results.each do |url, result|
        if result.key?(:error)
          puts "  ✗ #{url}"
          puts "    #{result[:error]} (#{result[:type]})"
        end
      end
    end

    results
  end
end

# Example usage with various URLs (some will fail)
urls = [
  'https://jsonplaceholder.typicode.com/posts/1',  # Valid JSON API
  'https://www.google.com',                        # Valid HTML
  'https://httpstat.us/500',                       # Server error (for retry demo)
  'https://httpstat.us/404',                       # Client error (no retry)
  'https://nonexistent-domain-12345.com',          # DNS error
]

handler = NetworkRequestHandler.new(max_retries: 2, timeout: 5)

puts "Fetching multiple URLs with exception handling and retry logic:"
results = handler.fetch_multiple(urls)
```

## Method-Level Exception Handling

Ruby allows you to add rescue clauses directly to methods:

```ruby
def risky_method
  # method body
  perform_operation
rescue ArgumentError => e
  puts "Invalid argument: #{e.message}"
  return nil
rescue => e
  puts "Unexpected error: #{e.message}"
  raise  # Re-raise if you can't handle it
end

# This is equivalent to:
def risky_method
  begin
    perform_operation
  rescue ArgumentError => e
    puts "Invalid argument: #{e.message}"
    return nil
  rescue => e
    puts "Unexpected error: #{e.message}"
    raise
  end
end
```

## Exception Information and Backtrace

Ruby exceptions carry useful information for debugging:

```ruby
begin
  raise "Something went wrong!"
rescue => e
  puts "Exception class: #{e.class}"
  puts "Exception message: #{e.message}"
  puts "Backtrace:"
  puts e.backtrace.first(5)  # Show first 5 lines of backtrace

  # Full backtrace
  puts "\nFull backtrace:"
  e.backtrace.each_with_index do |line, index|
    puts "  #{index}: #{line}"
  end
end
```

## Catch and Throw (Non-Local Exits)

Ruby provides `catch` and `throw` for non-local exits, which are different from exceptions:

```ruby
def find_user(users, target_name)
  catch(:found) do
    users.each do |user|
      user[:friends].each do |friend|
        throw(:found, friend) if friend[:name] == target_name
      end
    end
    nil  # Not found
  end
end

# Usage
users = [
  { name: "Alice", friends: [{ name: "Bob" }, { name: "Charlie" }] },
  { name: "David", friends: [{ name: "Eve" }, { name: "Frank" }] }
]

result = find_user(users, "Charlie")
puts result ? "Found: #{result[:name]}" : "Not found"
```

## Common Exception Types and When to Use Them

### Built-in Exceptions:

```ruby
# ArgumentError - Invalid arguments
def divide(a, b)
  raise ArgumentError, "Arguments must be numbers" unless a.is_a?(Numeric) && b.is_a?(Numeric)
  raise ZeroDivisionError, "Cannot divide by zero" if b.zero?
  a / b
end

# TypeError - Wrong type
def process_array(arr)
  raise TypeError, "Expected Array, got #{arr.class}" unless arr.is_a?(Array)
  arr.map(&:to_s)
end

# RuntimeError - General runtime errors
def validate_state
  raise "Invalid application state" unless valid_state?
end

# IOError - Input/output errors
def read_config
  raise IOError, "Config file is corrupted" unless valid_config_format?
end
```

## Best Practices for Exception Handling

### 1. Be Specific with Exception Types

```ruby
# Bad - too generic
begin
  operation
rescue
  puts "Something went wrong"
end

# Good - specific handling
begin
  operation
rescue ArgumentError => e
  puts "Invalid input: #{e.message}"
rescue IOError => e
  puts "File operation failed: #{e.message}"
rescue => e
  puts "Unexpected error: #{e.message}"
  raise  # Re-raise if you can't handle it properly
end
```

### 2. Don't Ignore Exceptions

```ruby
# Bad - silently ignoring errors
begin
  risky_operation
rescue
  # Silent failure
end

# Good - at least log the error
begin
  risky_operation
rescue => e
  logger.error "Operation failed: #{e.message}"
  # Handle appropriately or re-raise
end
```

### 3. Use Custom Exceptions for Domain Logic

```ruby
class BankAccount
  class InsufficientFundsError < StandardError
    attr_reader :requested_amount, :available_balance

    def initialize(requested, available)
      @requested_amount = requested
      @available_balance = available
      super("Insufficient funds: requested #{requested}, available #{available}")
    end
  end

  def withdraw(amount)
    raise InsufficientFundsError.new(amount, @balance) if amount > @balance
    @balance -= amount
  end
end
```

### 4. Always Clean Up Resources

```ruby
def process_file(filename)
  file = File.open(filename)
  begin
    # Process file
    process_data(file.read)
  ensure
    file.close if file
  end
end

# Or better, use blocks that auto-close
def process_file(filename)
  File.open(filename) do |file|
    process_data(file.read)
  end  # File automatically closed
end
```

## Key Takeaways

- Use specific exception types rather than generic rescue clauses
- Create custom exceptions for domain-specific errors
- Always clean up resources using `ensure` or block syntax
- Don't ignore exceptions - at minimum, log them
- Use retry logic for transient failures (network, database)
- Provide meaningful error messages for debugging
- Re-raise exceptions you can't handle properly
- Use `catch/throw` for control flow, not error handling

Exception handling is crucial for building robust Ruby applications. By understanding the exception hierarchy, using appropriate rescue strategies, and following best practices, you can create applications that gracefully handle errors and provide excellent user experiences even when things go wrong.

{% include inarticle-adsense.html %}
