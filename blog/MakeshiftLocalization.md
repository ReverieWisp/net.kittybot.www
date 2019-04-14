Makeshift localization
---
Localization is the act of taking content, usually text in applications, and changing it for a given place, and it's useful for application customization. While more conventional examples may be translating an application from english to spanish, localization is useful because it can allow you to tailor for individual situations as well. For this chatbot, my goal was to allow for customizing specific responses in different environments or servers.

A small selection of projects I've been on previously required localization, the most extensive being a smaller IDE. That system was written with Electron and subsequently JavaScript, so it could use a bunch of constant variables that could be swapped out while it was running. In practice, this looked like:

```javascript
// ExampleFile.js
alert(Strings.POPUP_OUTPUT_IF_CODE_COMPILED);
```
```javascript
// Strings.js
var const POPUP_OUTPUT_IF_CODE_COMPILED = "The code compiled!";
```

And you know what? This was just fine, and we ended up using it to localize into 52 different languages successfully, but it wasn't without its issues. Code like this suffers from a few things though, and can't be applied as easily to something like a Java Chatbot. JavaScript is an interpreted language. This means it doesn't need to be compiled, so `Strings.js` could be swapped out for different languages on the fly. Java is reduced to bytecode tho, so immediately this becomes less convenient in a compiled application when dealing with lots of strings. The other issue is that I'd defined a constant that was extremely verbose, so I'd almost written in a reasonable default, or may as well have.

With pretty much just these two things in mind, I slapped together a prototype for what I wanted to do using the ini file format for key-value pair dumping with sections [ini4j](http://ini4j.sourceforge.net/) that worked by scraping java source files for a function call then logging the arguments, and dumping them to a file. This could then be read in and translated on the fly with the contents of the localization file after the fact. In use, it looked something like this:

```java
// ExampleFile.java
System.out.println(Localizer.Stub("Defaulted string"));
System.out.println(Localizer.Stub("Localized string"));
```
```
// localizations.ini
[ExampleFile]
Defaulted string =
Localized string = Localized text here
```

Behavior is as follows: Sections are defined simply for readability in the ini file. The ini key-value pairs are read into maps in the application, where the ini key is the map key, and the value is the ini value if specified, or the ini key if left unspecified. The two map pairs of the above localizations.ini file would be: 

```
// Some lookup somewhere...
[Key: "Defaulted String", Value: "Defaulted String"],
[Key: "Localized string", Value: "Localized text here"]
```

When `Localizer.Stub` is called during program execution, the key is looked up and the value is returned.

With this system, the localizer stubs need to be collected before the source is compiled and the files become inaccessible (granted, you could unzip the jar but that's a bit much). This can be done in a pre-compilation step tho, either manually or automatically. Since I'd eventually like the chatbot's server to rebuild itself when changes are pushed to the master branch on github, having an extra step triggered by command line or something is not out of the realm of possibility. Just to make it easier, I tweaked it so rebuilt ini files don't override existing key-value pairs they find.

Once the concept was proven I moved on to making the actual thing and wrapping up the process! To help handle more edge cases and because some of our strings contain slashes and colons, and all sorts of characters that aren't valid in correctly formatted ini files, I ended up parsing the localization file (renamed to localization.config) manually. I wrote the utility as a standalone class, making it accept colons, newlines, white-space, etc, in keys and values, and dubbed the structure the `TaggedPairStore`. It's not the cleanest way to store and parse information, but it gets the job done and can be leveraged nicely to perform what we needed for a first pass. 

After a day of in-project use, we quickly realized that it was impractical to have the default string be anything but an easy to read define/const that provided some context for where in code it was. We kept our localization structures and internal behavior the same as before - our localization stubs now looked somthing like `Localizer.Stub("DiceRollInfo")` so that outside of the file, we had the context for what the string was going to be, and the default would be more obvious. As it currently stands, `TaggedPairStore.java`, `LocBase.java`, and `LocStrings.java` in the source contain the  full implementation and use case of what we have!

Going forward, we're looking to investigate leveraging the section labels in the config files more like an ini files so we can have multilpe keys at a global level, just not per tag. We're also looking into live reloading the files if they're edited to prevent needing to relaunch the bot. Hopefully we can get this in at a later time! 

Happy coding!
