# File Path Traversal, Validation of Start of Path

**Lab:** Exploiting insufficient validation of the beginning of a file path allows path traversal outside the intended directory and access to sensitive files.

**Category:** File Path Traversal

**Difficulty:** Practitioner

![](Path%20traversal/Screenshots/Screenshot_20260912_232701.png)

The goal of this lab is to retrieve the contents of `/etc/passwd`.

## Identifying the Vulnerable Endpoint

First, we need to find an endpoint that uses a file path to retrieve the product image.

Click **View details** on any product and open the product image in a new tab. The image is loaded through an endpoint similar to:

```text
/image?filename=24.jpg
```

The `filename` parameter appears to contain the path of the file that the application is trying to retrieve.

Intercept this request using **Burp Suite** and send it to **Repeater** so that we can modify the `filename` parameter.
![](Path%20traversal/Screenshots/Screenshot_20260912_233207.png)
## Understanding the Filter

The lab description gives us an important clue: the application expects the supplied path to **start with a specific directory**, such as:

```text
/var/www/images/
```

So simply trying:

```text
/etc/passwd
```

will not work because the path does not start with the expected directory.

At first, this might seem like a good security check. However, the application is only checking the **beginning of the path** and is not properly preventing traversal sequences later in the path.

## Bypassing the Path Validation

Since the application expects the path to start with:

```text
/var/www/images/
```

we can satisfy that check and then use `../` to move back out of the directory.

Try the following payload:

```text
/var/www/images/../../../etc/passwd
```

The important part here is that the path starts with the expected directory:

```text
/var/www/images/
```

so it passes the application's validation.

After that, the `../` sequences move up the directory structure.

Conceptually:

```text
/var/www/images/
        ../
        ../
        ../
```

moves from:

```text
/var/www/images/
```

back through the parent directories until we reach the filesystem root.

The remaining path:

```text
etc/passwd
```

then points to:

```text
/etc/passwd
```

## Exploiting the Vulnerability

Modify the `filename` parameter in Burp Suite Repeater:

```text
/image?filename=/var/www/images/../../../etc/passwd
```

Send the request.

The application accepts the path because it begins with the expected directory, but the traversal sequences allow us to escape that directory.

The response contains the contents of:

```text
/etc/passwd
```

![](Path%20traversal/Screenshots/Screenshot_20260912_233323.png)

The lab is therefore solved.

## Conclusion

The vulnerability exists because the application validates only that the supplied path **starts with the expected directory**.

Although the path begins with:

```text
/var/www/images/
```

the attacker can append traversal sequences such as:

```text
../../../
```

to escape the intended directory and access files elsewhere on the system.

The key idea is:

```text
/var/www/images/../../../etc/passwd
        ↓
passes "starts with /var/www/images/" check
        ↓
../../../ moves outside the intended directory
        ↓
/etc/passwd
```

This demonstrates why checking only the prefix of a user-supplied path is not sufficient. The application should properly canonicalize and validate the final resolved path before accessing the file.