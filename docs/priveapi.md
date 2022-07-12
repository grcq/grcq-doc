# PriveAPI

> ## Getting Started

Firstly, you will need to add the JAR file to your dependencies and add it to your plugins folder on your server.
To add the JAR file, create a folder and name it whatever you want. I highly recommend to name it <code>libs</code>.
Then go to your pom.xml and add the dependency like this:

**Maven**
```xml
<dependency>
    <groupId>cf.grcq</groupId>
    <artifactId>PriveAPI</artifactId>
    <version>1.0</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/libs/PriveAPI-1.0.jar</systemPath>
</dependency>
```

**Gradle**
```gradle
dependencies {
    compile files("$System.env.JAVA_HOME/libs/PriveAPI-1.0.jar)
}
```

When you have added the dependency, you may now reload your project.

> ## Creating Commands

Firstly, you need to create a class. You do not need to extend or implement it with anything.
Now do the following:
- Create a method with the `@Command` annotation. The method must be static.
- Fill out the annotation values. The first name you put is the main name, rest is aliases.

Now you could have something like this:
```java
@Command(names = {"teleport", "tp"}, permission = "mycommand.teleport", async = true)
public static void teleport() {

}
````

Now you need to add the parameters for your method. If you want to allow console and players to execute the command, put the first parameter as `CommandSender`, or if you want to only allow players to execute the command, put the first parameter as `Player`.
As of now, you could add parameters (depending on what command you make). If you want the first argument to be a player, you need to add `@Param(name = "target") Player target` as a second parameter. If you want to make it by default the executor, add `defaultValue = "@p"` to the Param annotation. There are limited parameters, so you cannot put whatever class as a parameter. Now you could have something like this:
```java
@Command(names = {"teleport", "tp"}, permission = "mycommand.teleport", async = true)
public static void teleport(Player player, @Param(name = "target") Player target) {

}
```

Now all you need to do is make the code that will be executed following the command.
You can register the commands in 2 different ways. Go to your main class, now you can register the command either manually or make it automatic.
- To manually add it, you'd need to do `CommandHandler.registerClass(this, MyCommand.class);` in onEnable().
- To automatically add commands, you'd need to do `CommandHandler.registerAll(this);` in onEnable().

> ## Creating Sub Commands 

If you want to have sub-commands, you still need a method to register the main command. You can make the main command execute something or make it show usage message.
To make it send the usage message. All you need to have is a static method with only the `CommandSender` or `Player` as a parameter, and the annotation would be `@Command(names = "core", sendUsage = true)`. Else, you don't need to add `sendUsage = true`in the annotation if you want to execute a different command via the main command.

Finally, you just need to do the same as you create a normal command, but the names has a space with the sub-command.
Example: `@Command(names = "core help")`

You would be having this as of now:
```java
// Main command, required to do this to register the command.
@Command(names = "core")
public static void core(CommandSender sender) {}

// Sub-command for the command "core".
@Command(names = "core help")
public static void coreHelp(CommandSender sender) {
    // Code
}
```

> ## Creating Parameters for Commands

Firstly, create a new class that implements `ParameterType<YourClass>`. "YourClass" is the class you want the parameter to cast to.
Now do the following:
- Implement all the methods, you can also implement the `tabComplete(Player player, String source)` method for tab-completion.
- Create your way of getting the class with the variables.
- Make it send a message if the transforming returns null.

Example:
```java
public class UserParameterType implements ParameterType<User> {

    @Override
    public User transform(CommandSender sender, String source) {
        User user = Main.getInstance().getUser(source);
        if (user != null) return user;

        sender.sendMessage("Could not find user '" + source + "'.")
        return null;
    }

    @Override
    public List<String> tabComplete(Player player, String source) {
        List<String> completion = new ArrayList<String>();
        Bukkit.getOnlinePlayers().forEach(player -> completion.add(player.getName()));

        return completion;
    }

}
```

After you have created your parameter, register it on your onEnable() method by doing:
`CommandHandler.registerParameter(MyClass.class, new MyParameterType());`
For me it would be `CommandHandler.registerParameter(User.class, new UserParameterType());`.

Now you can use the parameter in your command like `@Param(name = "name for parameter") MyClass myClass`.