---
layout: post
title: Overriding Content-Disposition and Content-Type on Google Cloud Storage and Amazon S3
description: How to override the Content-Disposition and Content-Type of Google Cloud Storage and Amazon S3 HTTP responses
keywords: Google, Amazon, storage, cloud, AWS, S3, Content-Type, Content-Disposition, response-content-type, response-content-disposition
---

I have been struggling to override the Content-Disposition and Content-Type 
headers of Google Cloud Storage and Amazon S3 responses. 
As you might know both S3 and 
Cloud Storage let you generate signed URLs 
(possibly set to expire in a specified amount of time) 
to access stored objects. Signed URLs are often used to grant 
non-authenticated users temporary access to otherwise private objects.
When the user opens a signed URL you may want the browser to download the object 
rather than trying to render it. 
It might as well be the case that the name you identify the object with 
is not the name you want the user to see when downloading the file. 
The `Content-Disposition` header of a response, if set properly, can 
tell the browser to download the object and can even set a default filename 
in the SaveAs prompt.  

The question is: **how do I set the Content-Disposition header of S3 and Storage responses?**  

Digging into Amazon's and Google's documentation I found the 
`response-content-disposition` parameter: if added to a signed URL 
it overrides the `Content-Disposition` header of the response allowing, 
for instance, to show a SaveAs prompt when accessing the object. 
Given a signed URL, you can override 
the Content-Disposition to download the file as `newfile.ext2`:  

```
<signed URL>&response-content-disposition="attachment; filename=newfile.ext2"
```

More details on Content-Disposition header's format can 
can be found [here](http://www.iana.org/assignments/cont-disp/cont-disp.xhtml).
If you are interested, it is also possible to override the `Content-Type` header 
of a response by providing the `response-content-type` parameter. 

Please notice that both `response-content-disposition` and 
`response-content-type` parameters must not be part of URL's signature, 
they can be simply appended once the URL has been signed.
