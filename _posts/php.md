<p>]---
title: "PHP"
layout: "post"
color: "fff"
background: "757eb1"
author: "Davey Shafik"
author_url: "http://daveyshafik.com"
author_twitter: "dshafik"
gravatar: "fee39f0c0ffb29d9ac21607ed188be6b"
sponsor: "Engine Yard"
sponsor_url: "https://www.engineyard.com"
sponsor_twitter: EngineYard</p>

<h2>sponsor_image: "http://securepasswords.info/themes/dshafik/securepasswords.info/assets/images/engineyard.png"</h2>

<p>With PHP 5.5, a new <a href="http://php.net/password">password extension</a> was added to that makes password security easy by default.</p>

<h2>Installation</h2>

<p>The password extension is enabled by default in PHP 5.5+. If you are not yet using PHP 5.5, you can use the <a href="https://github.com/ircmaxell/password_compat">password_compat</a> library, as far back as PHP 5.3.17.</p>

<p>To install password_compat, you can use composer:</p>

<pre><code class="sh">$ composer require ircmaxell/password_compat
</code></pre>

<h2>Usage</h2>

<p>The implementation consists of four functions:</p>

<ul>
<li><a href="http://php.net/password_hash"><code>password_hash()</code></a>
Create a new password hash</li>
<li><a href="http://php.net/password_verify"><code>password_verify()</code></a>
Confirm a given password matches the hash</li>
<li><a href="http://php.net/password_needs_rehash"><code>password_needs_rehash()</code></a>
Check if a password meets the desired hash settings (algorithm, cost)</li>
<li><a href="http://php.net/password_get_info"><code>password_get_info()</code></a>
Returns information about the hash such as algorithm and cost</li>
</ul>

<h3>Hashing a Password</h3>

<p>To hash a password, do the following:</p>

<pre><code class="php">&lt;?php
$password = password_hash($_POST['password'], PASSWORD_DEFAULT, ['cost' =&gt; 11]);
?&gt;
</code></pre>

<p>You may notice the second argument of <code>PASSWORD_DEFAULT</code>. This will automatically select the best algorithm at the time, as decided by PHP. Currently it is bcrypt, but this option gives you automatic forward compatible security, should bcrypt be compromised or become ineffectual in some way.</p>

<p>Additionally, we supply a third argument with the <code>cost</code>. The cost is what determines[a] how long the password takes to generate. It defaults to 10, however this is just a general guideline and your hardware will determine what makes more sense. The difference between an Amazon EC2 m3.medium and a c3.8xlarge is obviously going to have a significant impact on how long it takes to generate a hash.</p>

<h3>Verifying a Password</h3>

<p>To verify a password, we do:</p>

<pre><code class="php">&lt;?php
$user = User::findByUsername($username);
if (password_verify($_POST['password'], $user-&gt;password)) {
     // Password matches
} else {
     // Password does not match
}
?&gt;
</code></pre>

<h3>Keeping Your Users Secure</h3>

<p>The last part of the puzzle is ensuring that passwords are kept up to date. We can do this in two ways. One by using <code>password_get_info()</code> to check the algorithm and cost against our current settings. Or two, by using the built-in function <code>password_needs_rehash()</code> to check the hash against the settings automatically. By checking this on login, we can automatically rehash and store their password using the latest settings as supplied by <code>PASSWORD_DEFAULT</code>, or a change in the cost option:</p>

<pre><code class="php">&lt;?php
$user = User::findByUsername($username);
if (password_verify($_POST['password'], $user-&gt;password)) {
    if (password_needs_rehash($user-&gt;password, PASSWORD_DEFAULT, ['cost' =&gt; 12])) {
        $user-&gt;updatePassword($_POST['password']);
    }
} else {
     // Password does not match
}
?&gt;
</code></pre>

<p>The <code>password_needs_rehash()</code> function will return <code>true</code> for MD5 and SHA-1 hashes also. This means that you can check prior to <code>password_verify()</code> and use your legacy mechanism for verification instead. Once verified, you can then update the password using <code>password_hash()</code>. In this way you can roll out security updates for your users whilst avoiding mass password resets that would inconvenience your users. Though, if you are using (unsalted, especially) MD5 or SHA-1, it’s probably a good idea to do a mass reset anyway.</p>
