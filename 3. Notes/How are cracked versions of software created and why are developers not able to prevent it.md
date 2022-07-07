

# How are cracked versions of software created and why are developers not able to prevent it
Created:  [[2022-07-07]]
Tags: #fleeting 

---
Cracked versions of software are created with the use of [[debuggers]]. (A debugger is a special type of software that lets programmers deconstruct their software into its constituent parts for the purpose of finding bugs and thus de-bugging. 

Additionally debuggers can be used for reverse-engineering, or to see what is inside the software, to learn its logic. 

The latter method is used mostly by malware researchers to study what malware (or computer viruses) do on-the-inside. But it can be also used by an attacker to "crack" (or bypass) legal software registration, or at times, to alter normal behavior of software, for instance by injecting a malicious code into it.)

For the sake of this example, I will assume that the software that is being "cracked" was compiled into a native code, and is not a .NET or a JavaScript based application. (Otherwise it will be somewhat trivial to view its source code.) The compiled native code is a bit more tricky "beast" to study. (Native means that the code executes directly by the CPU, GPU, or other hardware.)

So let's assume that the goal of an attacker is to bypass the registration logic in the software so that he or she doesn't have to pay for it. (Later for lolz, he or she may also post such "crack" on some shady online forum or on a torrent site so that others can "use" it too and give him or her their appreciation.)

For simplicity let's assume that the original logic that was checking for the software registration was written in C++ and was something similar to the following code snippet:

![](https://qph.fs.quoracdn.net/main-qimg-aa9899f92cb2438b4bc310c3095afaec)

In this code sample "`RegistrationName`" and "`RegistrationCode`" are special strings of text that a legitimate software user will receive after paying for the license. (The name is usually that person's actual name or their email address, and the code is some string of unique/special characters that is tied to the name.)

In the logic above, the function named "`isRegistrationCodeGood()`" will check if "`RegistrationName`" and "`RegistrationCode`" are accepted using some proprietary method. If they are, it will return `true`. Otherwise `false`. That outcode will dictate which branch (or scope) the execution will follow.

So the logic above will either show that registration failed and quit:

![](https://qph.fs.quoracdn.net/main-qimg-f1cc891fa17f7f2c3084535cadd16b65)

Or, if the registration code and name matched, it will save the registration details in persistent storage (such as the File System or System Registry) using the function named "`rememberRegistrationParameters()`" and then display the message thanking the user for registering:

![](https://qph.fs.quoracdn.net/main-qimg-aa797644ae46a16233df23e02ee1ca23)

A "cracker" will obviously want to achieve the second result for any registration code that he or she enters. But they have a problem. They do not have the C++ source code, part of which I showed above. (I hope not!)

So the only recourse for an attacker is to [disassemble](https://en.wikipedia.org/wiki/Disassembler "en.wikipedia.org") the binary code (that always ships with software in the form of `.exe` and `.dll` files on Windows, and mostly as `Unix executables` inside the `.app` packages on a Mac.) An attacker will then use a debugger to study the binary code and try to locate the registration logic that I singled out above.

Next you can see the flowchart for a snippet of code that I showed in C++, presented via a low-level debugger. Or, as the code will be read in the binary form after [compilation](https://en.wikipedia.org/wiki/Compiler "en.wikipedia.org"):

(For readability I added comments on the right with the names of functions and variables. They will not be present in the code that an attacker could see.)

![](https://qph.fs.quoracdn.net/main-qimg-7665f7f54b2683a41ba24ffe595fe450-pjlq)

(To understand what is shown above an attacker will have to have good knowledge of the Assembly language instructions for the native code.)

I also need to point out that having a disassembly snippet like the one above is the final result for an attacker. The main difficulty for him or her is to locate it among millions and millions of other similar lines of code. And that is their main challenge. Not many people can do it and that is why software "cracking" is a special skill.

---

So having found the code snippet above in the software binary file a "cracker" has two choices:

1) Modify (or patch) the binary.

2) Reverse-engineer the "`isRegistrationCodeGood()`" function and copy its logic to create what is known as a "KeyGen" or "Key Generator."

Let's review both:

The first choice is quite straightforward. Since an attacker got this far, he or she knows the Intel x64 Instruction Set quite well. So they simply change the conditional jump from "`jnz short loc_7FF645671430`" at the address `00007FF645671418` (circled in red in the screenshots) to unconditional jump, or "`jmp short loc_7FF645671430`". This will effectively remove any failed registration code entries and anything that the user types in will be accepted as a valid registration.

Also note that this modification can be achieved by changing just one byte in the binary code from `0x75` to `0xEB`:

![](https://qph.fs.quoracdn.net/main-qimg-3ab42f46394e7243d89511c91cc34a78-pjlq)

But this approach comes with a "price" of modifying the original binary file. For that an attacker needs to write his own "patcher" (or a small executable that will apply the modification that I described above.) The downside of this approach for an attacker is that patching an original executable file will break its digital signature, which may alert the end-user or the vendor. Additionally the "patcher" executable made by an attacker can be easily flagged and blocked by the end-user's antivirus software, or lead criminal investigators to the identity of the attacker.

The second choice is a little bit more tricky. An attacker will have to study "`isRegistrationCodeGood()`" function and copy it into his own small program that will effectively duplicate the logic implemented in the original software and let him generate the registration code from any name, thus giving any unscrupulous user of that software an ability to register it without making a payment.

Vendors of many major software products understand the potential impact of the second method and try to prevent it by requiring what is known as "authentication." This is basically a second step after registration, where the software submits registration name to the company's web server that returns a response back to the software of whether the code was legitimate or not. This is done by Microsoft when you purchase Windows (they call it "Activate Windows") and also by Adobe, and many other companies. This second step may be done behind-the-scenes on the background while the software is running, and will usually lead to cancellation of prior registration if it was obtained illegally.

---

So now you know how software is "cracked".

Let me answer why it is not possible to prevent it. It all boils down to the fact that any software code needs to be read either by CPU (in case of a binary native code) or by an [interpreter](https://en.wikipedia.org/wiki/JavaScript_engine "en.wikipedia.org") or a [JIT compiler](https://en.wikipedia.org/wiki/Just-in-time_compilation "en.wikipedia.org") (in case of JavaScript or .NET code.) This means that if there's a way to read/interpret something, no matter how complex or convoluted it is, an attacker with enough knowledge and persistence will be able to read it as well, and thus break it.

There is an argument though that cloud-based software is more secure, which is true, since its (binary) code remains on the server and end-users do not have direct access to it. And even though cloud-based software is definitely the future, it has some major drawbacks that will never allow it to fully replace your conventional software. To name just a few:

-   Not everyone has an internet connection, or is willing to upload their data online. Additionally someone’s internet connection can be very expensive or too slow to make the software run very laggy.
-   Then there’s a question of distributed computing. For instance, Blizzard Entertainment would never make “World of Warcraft” to fully run on their servers due to immense computational resources needed to render every single scene for every player they have. Thus it is in their best interest to let each individual user’s computer to do the rendering instead.

As a software developer myself, I obviously don't like when people steal software licenses. But I have to accept it and live with it. The good news is that there are not that many people who are willing to go extra mile and search for a cracked version of software. The main problem for those who do, is that by downloading a patched executable, or an attacker's KeyGen or a Patcher, they are effectively "trusting" him or her not to put anything "nasty" into it that was not "advertised on the package" (stuff like trojans, malware, or keyloggers.) So the question for those people becomes -- is it worth the cost of the software license to potentially infect your system with a nasty virus?

On the other side of the equation, some developers react very negatively to any attempts to steal their software licenses. (I was there too.) They try to implement all kinds of countermeasures -- anything from tricking reverse-engineers, to adding booby traps in the code that may do something nasty if the code detects that it is being debugged, to obfuscating or scrambling the code, to enforcing all kinds of convoluted [DRM](https://en.wikipedia.org/wiki/Digital_rights_management "en.wikipedia.org") schemes, to blocking users from certain countries. I personally try to stay away from all of those measures. And here's why:

A) Any kind of anti-reverse-engineering tactics could be bypassed by an attacker with enough persistence. So why bother and waste my time when I can invest that time into adding something useful to my software that will make it more productive for legitimate users?

B) Some [code packers](https://en.wikipedia.org/wiki/Executable_compression "en.wikipedia.org") could create false positives with antivirus software, which is obviously not good for marketing of that software. It also creates unnecessary complexity for the developer to debug the software.

C) Adding booby traps in the code can also “misfire” on your legitimate users, which will really infuriate them and can even lead to lawsuits.

D) Any [DRM](https://en.wikipedia.org/wiki/Digital_rights_management "en.wikipedia.org") scheme will probably catch some 100 illegal users and greatly inconvenience 10,000 legitimate ones. So why do it to your good customers?

E) Our statistics show that about 75% of all illegal licenses come from China, Russia, Brazil, to name the worst offenders. (I also understand that the reason may be much lower incomes that people have in those countries.) The main issue for us though was the fact that if we enforce our DRM or add some strong registration authentication, many people that wanted to bypass our registration would simply use a stolen credit card number. And we had no control over it. Our system will use it to send them a legitimate license only to have the payment bounce in weeks time. As a result we would lose the money that were paid for the license, plus the credit card company will impose an additional chargeback fee to our account, which may range from $0.25 to $20 per bad purchase on top of the license cost.












### References
1. 