# Selenium Chrome HTTP Private Proxy

This plugin allows us to use a proxy with basic authentication with ChromeDriver and Selenium ([it's impossible currently](http://docs.seleniumhq.org/docs/04_webdriver_advanced.jsp#using-a-proxy)).

This is a Java implementation of the plugin that can be found here: https://github.com/RobinDev/Selenium-Chrome-HTTP-Private-Proxy

## How to use it

Add this method, optionally change the directory where the plugin files will be saved, currently specefied to save in: ```user.home\.application\auth\```

```java
    public File getExtension(String host, String port, String username, String password) throws Exception {
        String pluginDir = System.getProperty("user.home") + System.getProperty("file.separator") + ".application" + System.getProperty("file.separator") + "auth" +  System.getProperty("file.separator");
        String fileName = pluginDir + host+"-"+port+"-"+username+"-"+password+ ".zip";
        File extensionFile = new File(fileName);
        if (extensionFile.exists()) {
            return extensionFile;
        }
        FileOutputStream fout = new FileOutputStream(extensionFile);
        ZipOutputStream zout = new ZipOutputStream(fout);
        ZipEntry manifestFile = new ZipEntry("manifest.json");
        zout.putNextEntry(manifestFile);
        String manifestContent = new String("{\n" +
                "    \"version\": \"1.0.0\",\n" +
                "    \"manifest_version\": 2,\n" +
                "    \"name\": \"Chrome Proxy\",\n" +
                "    \"permissions\": [\n" +
                "        \"proxy\",\n" +
                "        \"tabs\",\n" +
                "        \"unlimitedStorage\",\n" +
                "        \"storage\",\n" +
                "        \"<all_urls>\",\n" +
                "        \"webRequest\",\n" +
                "        \"webRequestBlocking\"\n" +
                "    ],\n" +
                "    \"background\": {\n" +
                "        \"scripts\": [\"background.js\"]\n" +
                "    },\n" +
                "    \"minimum_chrome_version\":\"22.0.0\"\n" +
                "}\n");
        byte[] manifestData = manifestContent.getBytes();
        zout.write(manifestData, 0, manifestData.length);
        zout.closeEntry();
        ZipEntry scriptFile = new ZipEntry("background.js");
        zout.putNextEntry(scriptFile);
        String scriptContent = new String("\n" +
                "\n" +
                "var config = {\n" +
                "        mode: \"fixed_servers\",\n" +
                "        rules: {\n" +
                "          singleProxy: {\n" +
                "            scheme: \"http\",\n" +
                "            host: \"%proxy_host\",\n" +
                "            port: parseInt(%proxy_port)\n" +
                "          },\n" +
                "          bypassList: [\"foobar.com\"]\n" +
                "        }\n" +
                "      };\n" +
                "\n" +
                "chrome.proxy.settings.set({value: config, scope: \"regular\"}, function() {});\n" +
                "\n" +
                "function callbackFn(details) {\n" +
                "    return {\n" +
                "        authCredentials: {\n" +
                "            username: \"%username\",\n" +
                "            password: \"%password\"\n" +
                "        }\n" +
                "    };\n" +
                "}\n" +
                "\n" +
                "chrome.webRequest.onAuthRequired.addListener(\n" +
                "            callbackFn,\n" +
                "            {urls: [\"<all_urls>\"]},\n" +
                "            ['blocking']\n" +
                ");");
        scriptContent = scriptContent.replaceAll("%proxy_host", host);
        scriptContent = scriptContent.replaceAll("%proxy_port", port);
        scriptContent= scriptContent.replaceAll("%username", username);
        scriptContent = scriptContent.replaceAll("%password", password);
        byte[] scriptData = scriptContent.toString().getBytes();
        zout.write(scriptData, 0, scriptData.length);
        zout.closeEntry();
        zout.close();
        return extensionFile;
    }
```

Now add the extension to the ChromeDriver instance like this

```java
        DesiredCapabilities capabilities = DesiredCapabilities.chrome();
        ChromeOptions options = new ChromeOptions();
        options.addExtensions(getExtension("host", "port", "username", "password"));
        capabilities.setCapability(ChromeOptions.CAPABILITY, options);
        ChromeDriver driver = new ChromeDriver(capabilities);
        driver.get("http://www.whatsmyip.org/");
```
