# File Path Traversal, Validation of File Extension with Null Byte Bypass

**Lab:** Exploiting a null byte to bypass file extension validation allows path traversal and unauthorized access to sensitive files.

**Category:** File Path Traversal

**Difficulty:** Practitioner
![](Path%20traversal/Screenshots/Screenshot_20260912_233827.png)

The goal of this lab is to retrieve the contents of `/etc/passwd`.
## Identifying the Vulnerable Endpoint

First, we need to find an endpoint that uses a filename to retrieve the product image.

Click **View details** on any product and open the product image in a new tab. The image is loaded through an endpoint similar to:

```text
/image?filename=33.jpg
```

The `filename` parameter appears to control which file the application retrieves.

Intercept the request using **Burp Suite** and send it to **Repeater** so that we can modify the `filename` parameter.
![](Path%20traversal/Screenshots/Screenshot_20260912_233543.png)
## Understanding the Filter

The lab description gives us an important clue: the application checks that the supplied filename **ends with the expected file extension**.

For example, if the application expects an image file, it may require the filename to end with:

```text
.png
```

or:

```text
.jpg
```

A normal path traversal payload such as:

```text
../../../etc/passwd
```

would fail this validation because it does not end with the expected image extension.

We therefore need to find a way to make the filename appear to have a valid extension during the validation step, while still causing the application to access `/etc/passwd`.

## Bypassing the Extension Check

The key to this lab is the **null byte**, represented in a URL as:

```text
%00
```

We can append the expected file extension after the null byte:

```text
../../../etc/passwd%00.png
```

The complete request becomes:

```text
/image?filename=../../../etc/passwd%00.jpg
```

The important part is the position of `%00`:

```text
../../../etc/passwd%00.jpg
                    ↑
              null byte
```

The application sees `.png` at the end of the supplied input, so the filename can pass the extension validation.

However, when the value is later processed by a component that treats the null byte as a string terminator, everything after `%00` is ignored.

Conceptually, the filename can therefore be interpreted as:

```text
../../../etc/passwd
```

instead of:

```text
../../../etc/passwd.jpg
```

## Exploiting the Vulnerability

Modify the `filename` parameter in **Burp Suite Repeater**:

```text
/image?filename=../../../etc/passwd%00.jpg
```

Send the request.

The application accepts the filename because it ends with the expected `.png` extension, but the null byte causes the extension to be effectively discarded during subsequent processing.

The resulting path traversal allows the application to retrieve:

```text
/etc/passwd
```

![](Path%20traversal/Screenshots/Screenshot_20260912_233636.png)

The response contains the contents of `/etc/passwd`, so the lab is solved.

## Conclusion

The vulnerability exists because the application performs **file extension validation before safely handling the null byte**.

The payload:

```text
../../../etc/passwd%00.png
```

works because it combines two things:

```text
../../../etc/passwd
        ↓
Path traversal
```

and:

```text
%00.png
   ↓
Null byte followed by a valid extension
```

The extension satisfies the application's validation, while the null byte can cause the later file-handling logic to ignore everything after it.

This demonstrates why validating a filename by simply checking its extension is not sufficient. User-controlled paths should be safely canonicalized and validated before the file is accessed.