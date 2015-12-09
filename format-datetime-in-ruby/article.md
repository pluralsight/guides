In Ruby, you have to format or convert time from one format to other many a times. You can use library method **strftime** available to perform formatting operations.

Syntax of strftime: **strftime( format )**

Returns: Result in requested format First of all, let us create a new variable in which we get current time to be formatted in various formats,

``` ruby
t = Time.now > 2013-09-12 22:49:27 +0530
```
Now, we have variable t holding current time value, now below are the examples of,

### Most Commonly used DateTime Formats

| Code | Output | Description |
|------------------------------
| t.strftime("%H") | => "22" | # Gives **Hour** of the time **in 24 hour** clock format |
| t.strftime("%I") | => "10" | # Gives **Hour** of the time **in 12 hour** clock format |
| t.strftime("%M") | => "49" | # Gives **Minutes** of the time |
| t.strftime("%S") | => "27" | # Gives **Seconds** of the time |
| t.strftime("%Y") | => "2013" | # Gives **Year** of the time |
| t.strftime("%m") | => "09" | # Gives **month** of the time |
| t.strftime("%d") | => "12" | # Gives **day** of month of the time |
| t.strftime("%w") | => "4" | # Gives **day of week** of the time |
| t.strftime("%a") | => "Thu" | # Gives name of ** week day** in **short** form of the |
| t.strftime("%A") | => "Thursday" | # Gives **week day** in **full** form of the time |
| t.strftime("%b") | => "Sep" | # Gives **month** in **short** form of the time |
| t.strftime("%B") | => "September" | # Gives **month** in **full** form of the time |
| t.strftime("%y") | => "13" | # Gives **year without century** of the time |
| t.strftime("%Y") | => "2013" | # Gives **year without century ** of the time |
| t.strftime("%Z") | => "IST" | # Gives **Time Zone** of the time |
| t.strftime("%p") | => "PM" | # Gives **AM / PM** of the time |

These are the almost all formats that are required.

### Combinations:

You can try these formats in combination too, For example,
``` ruby
t.strftime("%H:%M:%S") > "22:49:27"
```


### Code to Print All Formats

``` ruby
# Function to print strftime results
def print_strftime_formats(a,cur_date)
 a.each do |format|
   b = "%#{format}"
   output = cur_date.strftime(b)
   puts "t.strftime('#{b}'), =&gt; #{output}"
 end
end

a = ('a'..'z').to_a
A = ('A'..'Z').to_acur_date = Time.now

# calling and printing strftime results on current date

puts "DateTime: #{cur_date}n"

puts "nnFor a to z"
print_strftime_formats(a,cur_date)

puts "nnFor A to Z"
print_strftime_formats(A,cur_date)
```

You can refer code at [Github](https://gist.github.com/akshaymohite/8578315) OR [download](https://gist.github.com/akshaymohite/8578315/download) this code. The output of the above code is given below for the reference:

### For 'a' to 'z'

| Code | Output |
|----------------
| t.strftime('%a') | => Thu |
| t.strftime('%b') | => Jan |
| t.strftime('%c') | => Thu Jan 23 16:38:02 2014 |
| t.strftime('%d') | => 23 |
| t.strftime('%e') | => 23 |
| t.strftime('%f') | => %f # Not Useful |
| t.strftime('%g') | => 14 |
| t.strftime('%h') | => Jan |
| t.strftime('%i') | => %i # Not Useful |
| t.strftime('%j') | => 023 |
| t.strftime('%k') | => 16 |
| t.strftime('%l') | => 4 |
| t.strftime('%m') | => 01 |
| t.strftime('%n') | => # Not Useful |
| t.strftime('%o') | => %o |
| t.strftime('%p') | => PM |
| t.strftime('%q') | => %q |
| t.strftime('%r') | => 04:38:02 PM |
| t.strftime('%s') | => 1390475282 |
| t.strftime('%t') | => # Not Useful |
| t.strftime('%u') | => 4 |
| t.strftime('%v') | => 23-JAN-2014 |
| t.strftime('%w') | => 4 |
| t.strftime('%x') | => 01/23/14 |
| t.strftime('%y') | => 14 |
| t.strftime('%z') | => +0530 |

### For 'A' to 'Z'

| Code | Output |
|----------------
| t.strftime('%A') | => Thursday |
| t.strftime('%B') | => January |
| t.strftime('%C') | => 20 |
| t.strftime('%D') | => 01/23/14 |
| t.strftime('%E') | => %E # Not Useful |
| t.strftime('%F') | => 2014-01-23 |
| t.strftime('%G') | => 2014 |
| t.strftime('%H') | => 16 |
| t.strftime('%I') | => 04 |
| t.strftime('%J') | => %J # Not Useful |
| t.strftime('%K') | => %K # Not Useful |
| t.strftime('%L') | => 485 |
| t.strftime('%M') | => 38 |
| t.strftime('%N') | => 485141000 |
| t.strftime('%O') | => %O # Not Useful |
| t.strftime('%P') | => pm |
| t.strftime('%Q') | => %Q # Not Useful |
| t.strftime('%R') | => 16:38 |
| t.strftime('%S') | => 02 |
| t.strftime('%T') | => 16:38:02 |
| t.strftime('%U') | => 03 |
| t.strftime('%V') | => 04 |
| t.strftime('%W') | => 03 |
| t.strftime('%X') | => 16:38:02 |
| t.strftime('%Y') | => 2014 |
| t.strftime('%Z') | => IST |

So, you can tweak your format to give the datetime output required for your purpose. I can only hope that the formatting of datetime data will be easier for you to handle now.

