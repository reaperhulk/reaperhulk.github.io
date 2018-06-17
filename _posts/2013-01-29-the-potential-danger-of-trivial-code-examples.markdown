---
author: admin
comments: true
date: 2013-01-29 16:10:52+00:00
layout: post
slug: the-potential-danger-of-trivial-code-examples
title: The (Potential) Danger of Trivial Code Examples
wordpress\_id: 1902
tags:
- complaints dept
- php
- ruby
---

As programmers there are two things we appreciate most when attempting to accomplish something: documentation and examples. Unfortunately, for some classes of problems trivial examples take shortcuts that, if implemented in production code, cause performance problems and security flaws. Let's take a tour.[^1]



### SQL Example


Here's how to do a SQL query in PHP (3 ways!)

```php
$username = $_REQUEST['username'];
mysql_connect($server,$user,$password,$db);
mysql_query("SELECT * FROM my_table where username=$username");
//but that's deprecated as of PHP5.5...let's do it with mysqli
$dbconn = mysqli_connect($server,$user,$password,$db);
mysqli_query($dbconn,"SELECT * FROM my_table where username=$username");
//or maybe we want PDO
$pdo = new PDO('mysql:host=example.com;dbname=database', 'user', 'password');
$statement = $pdo->query("SELECT * FROM my_table where username=$username");

```

Done! You know how to build SQL queries from query parameters now! Or...maybe not. Obviously each of these examples is potentially vulnerable to SQL injection. There are quite a few ways to solve this, but sample code rarely mentions that this code is utterly unsafe for production use because that would be "needlessly complicated".



### Better SQL Example



```php
$username = $_REQUEST['username'];
//deprecated as of 5.5
mysql_connect($server,$user,$password,$db);
$safe_query = sprintf("SELECT * FROM my_table where username='%s'",mysql_real_escape_string($username));
mysql_query($safe_query);

$dbconn = mysqli_connect($server,$user,$password,$db);
$safe_query = sprintf("SELECT * FROM my_table where username='%s'",mysqli_real_escape_string($username));
mysqli_query($dbconn,"SELECT * FROM my_table where username=$username");

$pdo = new PDO('mysql:host=example.com;dbname=database', 'user', 'password');
$statement = $pdo->prepare("SELECT * FROM my_table where username=:username");
$statement->execute(array(':username' => $username));

```

There is definitely a higher cognitive load associated with figuring out what these snippets do, but it is not high enough to justify the problems caused by ignoring it. The people who most need training about the danger inherent in these methods are the ones who are Googling for examples.



### Network Socket Example


So you need to open a TCP socket? Here's one way in Ruby.

```ruby
require 'socket'
server = TCPServer.open(10000)
loop {
  client = server.accept
  client.puts "At the sound of the tone the current timestamp will be #{Time.now.to_i}"
  client.puts "Tone!"
  client.close
}

```

Go ahead, telnet to it and you'll see the message. Pretty cool right? Except you're bound to every interface on your machine.



### Better Network Socket Example



```ruby
require 'socket'
server = TCPServer.open("localhost",10000)
loop {
  client = server.accept
  client.puts "At the sound of the tone the current timestamp will be #{Time.now.to_i}"
  client.puts "Tone!"
  client.close
}

```

There, now we're binding to just a single interface. Many examples of this sort of trivial TCP server online neglect to mention that binding to all interfaces will implicitly make your server available to anyone with network access to your machine.[^2]



### Populating Views Example


Many popular MVC frameworks in Ruby share instance variables from their controllers into the views. Let's build a trivial example.

```ruby

class SomeController
  def method
    @my_data = "Hello World!"
  end
end

```


```html
<%= @my_data %>
```

Pretty slick! Unless your MVC doesn't auto-escape your instance variables (Rails 3+ does this by default, calm down). If it does not, then if your variable comes from user input you have a potential XSS vector! A better example depends on the available HTML sanitization methods that the framework you choose to use supplies, so this is perhaps an example of where the security of apparently generic code is in fact highly dependent on the framework it runs within.



### HMAC Example


Totally trivial: HMAC-SHA256 signatures in Ruby.

```ruby
require 'openssl'
key = "key"
data = "data to sign"
digest = OpenSSL::Digest::SHA256.new
puts OpenSSL::HMAC.hexdigest(digest,key,data)

```

Simple. Well, as long as you ignore [RFC 2104](http://www.ietf.org/rfc/rfc2104.txt) at least. If you look deeper (and why would you? The code above works great!) you'll find the following:


>
   The definition of HMAC requires a cryptographic hash function, which
   we denote by H, and a secret key K. We assume H to be a cryptographic
   hash function where data is hashed by iterating a basic compression
   function on blocks of data.   We denote by B the byte-length of such
   blocks (B=64 for all the above mentioned examples of hash functions),
   and by L the byte-length of hash outputs (L=16 for MD5, L=20 for
   SHA-1).  The authentication key K can be of any length up to B, the
   block length of the hash function.  Applications that use keys longer
   than B bytes will first hash the key using H and then use the
   resultant L byte string as the actual key to HMAC. In any case the
   minimal recommended length for K is L bytes (as the hash output
   length).

   The key for HMAC can be of any length (keys longer than B bytes are
   first hashed using H).  However, less than L bytes is strongly
   discouraged as it would decrease the security strength of the
   function.  Keys longer than L bytes are acceptable but the extra
   length would not significantly increase the function strength. (A
   longer key may be advisable if the randomness of the key is
   considered weak.)

   Keys need to be chosen at random (or using a cryptographically strong
   pseudo-random generator seeded with a random seed), and periodically
   refreshed.  (Current attacks do not indicate a specific recommended
   frequency for key changes as these attacks are practically
   infeasible.  However, periodic key refreshment is a fundamental
   security practice that helps against potential weaknesses of the
   function and keys, and limits the damage of an exposed key.)



Encryption and digital signatures are tricky business and the cryptographic primitives people use to accomplish their goals are just that, primitive. As a programmer, you will likely be required to implement some type of encryption and signing at some point. It is an unfortunate reality that unless you have some training, experience, or are fortunate enough to use great tools you will likely get it wrong. Even something simple like having insufficient key length for your chosen HMAC digest could have serious consequences and no example is going to tell you that.



### Better HMAC Example



```ruby
require 'openssl'
digest = OpenSSL::Digest::SHA256.new
key = OpenSSL::Random.random_bytes(digest.digest_length)
data = "data to sign"
puts OpenSSL::HMAC.hexdigest(digest,key,data)

```

Pretty simple change, but now the key is decent. In this case I've chosen to generate enough random bytes to cover the length of the chosen digest. Of course, you don't actually want to be generating a new key for every signature so the complexity will be higher since a "real world" implementation would generate the key out of band, save it to disk, then load it as required.



### Best HMAC Example


First generate a key with sufficient length and entropy

```ruby
require 'openssl'
digest = OpenSSL::Digest::SHA256.new
key = OpenSSL::Random.random_bytes(digest.digest_length)
File.open("/some/path",'w') { |f| f.write(key) }

```


Now HMAC your data

```ruby
require 'openssl'
digest = OpenSSL::Digest::SHA256.new
key = File.read("/some/path")
data = "data to sign"
puts OpenSSL::HMAC.hexdigest(digest,key,data)

```

Of course now this code can't be executed without making at least some modifications for your own environment. Damn. Moving on.



### Encryption Example


Here's a problem that's more subtle than HMAC. AES 256 (via Ruby OpenSSL::Cipher[^3]) using cipher block chaining (a very common[^4] [block cipher mode](http://en.wikipedia.org/wiki/Block_cipher_modes_of_operation)). Again we'll be using Ruby.

```ruby
require 'openssl'
enc = OpenSSL::Cipher::AES256.new(:CBC)
enc.encrypt
key = enc.random_key # let's avoid the pitfall from HMAC and generate a good key
ct = enc.update("my encrypted text is extremely important")
ct << enc.final
# and for completeness, here's a decryption
dec = OpenSSL::Cipher::AES256.new(:CBC)
dec.decrypt
dec.key= key
pt = dec.update(ct)
pt << dec.final
puts pt

```

There, a basic example of encryption and we're using a good key. Unfortunately CBC requires two inputs for safe encryption, not just one. We've neglected the initialization vector (IV) so OpenSSL sets it to a 16 byte string of null characters (\0). This is very bad (and extremely common).



### Better Encryption Example



```ruby
require 'openssl'
enc = OpenSSL::Cipher::AES256.new(:CBC)
enc.encrypt
key = enc.random_key # let's avoid the pitfall from HMAC and generate a good key
iv = enc.random_iv # we need this too!
ct = enc.update("this encrypted text is actually somewhat secure")
ct << enc.final
# and for completeness, here's a decryption
dec = OpenSSL::Cipher::AES256.new(:CBC)
dec.decrypt
dec.key= key
dec.iv= iv
pt = dec.update(ct)
pt << dec.final
puts pt

```

Once again the fixed version does not appear significantly more complex than the (admittedly confusing) original example, but upon closer inspection problems appear. The key and IV are both generated on a per execution basis. You **do** want to do this with an IV, but not with a key...



### Best Encryption Example


First generate a key

```ruby
require 'openssl'
enc = OpenSSL::Cipher::AES256.new(:CBC)
enc.encrypt
key = enc.random_key # let's avoid the pitfall from HMAC and generate a good key
File.open("/some/path",'w') { |f| f.write(key) }

```


Now encrypt/decrypt your data

```ruby
require 'openssl'
enc = OpenSSL::Cipher::AES256.new(:CBC)
enc.encrypt
key = File.read("/some/path")
iv = enc.random_iv
ct = enc.update("no one can ever read this")
ct << enc.final
# and for completeness, here's a decryption
dec = OpenSSL::Cipher::AES256.new(:CBC)
dec.decrypt
dec.key= key
dec.iv= iv
pt = dec.update(ct)
pt << dec.final
puts pt

```

Once again we've had to split our example into two separate bits of code. One is executed a single time and the other is the common case. We do have a per-encryption IV that needs to be saved so you can fully decrypt later, so don't forget about that! But what if you want to use a password rather than a pile of random bytes?



### Password-Based Encryption Example


We've learned our lesson from before and we will not be forgetting the IV.

```ruby
require 'openssl'
enc = OpenSSL::Cipher::AES256.new(:CBC)
enc.encrypt
key = "Testing1" # a password provided by the user perhaps
iv = enc.random_iv # we need this too!
ct = enc.update("customer secrets go here!")
ct << enc.final
# and for completeness, here's a decryption
dec = OpenSSL::Cipher::AES256.new(:CBC)
dec.decrypt
dec.key= key
dec.iv= iv
pt = dec.update(ct)
pt << dec.final
puts pt

```

No problem. The user is at fault for their own terrible password right? We just gave the block cipher the key, generated a proper IV, and now we're done. Well, not so fast. Encryption has tricked you yet again. What you really need is [PBKDF2](http://en.wikipedia.org/wiki/PBKDF2), part of [RFC 2898](http://www.ietf.org/rfc/rfc2898.txt). (What, you didn't know that? Guess that code sample you copied might have been kind of dangerous...)



### Better Password-Based Encryption Example



```ruby
require 'openssl'
enc = OpenSSL::Cipher::AES256.new(:CBC)
enc.encrypt
user_password = "Testing1"
key = OpenSSL::PKCS5.pbkdf2_hmac_sha1(user_password,"somesalt",1000,32)
enc.key= key
iv = enc.random_iv
ct = enc.update("my first ciphertext")
ct << enc.final
# and for completeness, here's a decryption
dec = OpenSSL::Cipher::AES256.new(:CBC)
dec.decrypt
dec.key= key
dec.iv= iv
pt = dec.update(ct)
pt << dec.final
puts pt

```

Well that's...substantially less understandable. What the hell are all those parameters in pbkdf2\_hmac\_sha1 anyway? And is it a good idea to use "somesalt" as a fixed string? (It might be okay, or it might be a serious problem depending on what you're trying to accomplish. Damned nuance!)



### Best Password-Based Encryption Example


Now things are getting complicated... Let's assume we care about having a random salt (pretty likely since if you're encrypting with user passwords you'll find that users are terrible about having distinct passwords!). [RFC 2898](http://www.ietf.org/rfc/rfc2898.txt) recommends we use a salt of minimum length 64-bits (8 bytes). It also recommends a minimum of 1000 iterations of the password derivation function. Of course, the document is from 2000 and computers are much faster now than they used to be...Let's choose 10000 iterations (somewhat arbitrarily).

```ruby
require 'openssl'
enc = OpenSSL::Cipher::AES256.new(:CBC)
enc.encrypt
user_password = "Testing1"
length = 32 # byte length of an AES-256 key (256/8)
iterations = 10000
salt = OpenSSL::Random.random_bytes(8)
key = OpenSSL::PKCS5.pbkdf2_hmac_sha1(user_password,salt,iterations,length)
enc.key= key
iv = enc.random_iv
ct = enc.update("secure password-based encryption ahoy")
ct << enc.final
# and for completeness, here's a decryption
dec = OpenSSL::Cipher::AES256.new(:CBC)
dec.decrypt
dec.key= key
dec.iv= iv
pt = dec.update(ct)
pt << dec.final
puts pt

```

There, a nice secure way to do this. Of course, now that we've gone this far down the rabbit hole we have the issue of how to store per-encryption salts such that you can decrypt the ciphertext at a later time, but that's an exercise left to the reader.[^5]



### Some Other Possible Examples


I'm getting a bit long-winded so I'll leave you with some more ideas for dangerous trivial examples. If you've got more don't hesitate to speak up!




  * NSURLConnection synchronously or doing significant processing in the main thread


  * Authentication cookies


  * Password storage (plaintext, simple hashes, global salt)


  * Temporary file creation


  * Reading a large file (past integer limits or just huge memory consumption and poor performance)




[^1]: It is not my intent to single out any particular language. Since each language has such a distinct ecosystem of docs/examples each language possesses strengths and weaknesses with regard to the examples I've provided.

[^2]: Quiet in the back iptables aficionados.

[^3]: The current Ruby documentation around [OpenSSL::Cipher](http://www.ruby-doc.org/stdlib-1.9.3/libdoc/openssl/rdoc/OpenSSL/Cipher.html) is actually quite thorough about the pitfalls of the various things I discuss here, but when Googling for something like "ruby aes encryption" you can easily be steered in a dangerous direction

[^4]: The wisdom of using CBC is itself questionable, but let's stick to outright mistakes for now

[^5]: I couldn't resist.
