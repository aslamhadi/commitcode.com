---
title: Upload image summernote editor to Azure Storage
date: "2016-10-30T22:12:03.284Z"
---

When you upload an image to Summernote editor, it will convert the image to base64 data. And base64 is larger than the original data size. So in this post, I will explain how to upload the picture to Azure Storage instead of converting to base64.

> Very roughly, the final size of Base64-encoded binary data is equal to 1.37 times the original data size + 814 bytes (for headers). <a href="https://en.wikipedia.org/wiki/Base64" target="_blank">Wikipedia</a>

### Prerequisite

Install and setup the <a href="/development/upload-picture-asynchronous-azure-storage/" >Azure Storage</a>.

### Controller

Create a controller to upload the picture to Azure Storage. It's simply get the request file, upload, that's it.

```
public class SummernoteController : Controller
{
    private readonly IAzureFileUploaderService _azureFileUploaderService;

    public SummernoteController(IAzureFileUploaderService azureFileUploaderService)
    {            
        _azureFileUploaderService = azureFileUploaderService;
    }

    [HttpPost]
    public async Task<ActionResult> ImageUpload()
    {            
        var files = Request.Files;
        var url = string.Empty;
        if (files.Count > 0)
        {
            url = await _azureFileUploaderService.UploadPhotoAsync(files[0], Global.Full, "summernote");
        }
        return Json(url);
    }
}
```

### Javascript

We'll be using `onImageUpload` from summernote. <a href="http://summernote.org/deep-dive/#onimageupload" target="_blank">Documentation</a>

```
$('#summernote').summernote({
    height: 200,
    onImageUpload: function (files) {
        sendFile(files[0], this);
    }
});
```

The `el` parameter is basically the summernote DOM element. We need this to update the insert the uploaded picture from Azure Storage.

```
function sendFile(file, el) {
    var formdata = new FormData();
    formdata.append(file.name, file);
    var xhr = new XMLHttpRequest();
    xhr.open('POST', '/summernote/ImageUpload/');
    xhr.send(formdata);
    xhr.onreadystatechange = function () {
        if (xhr.readyState == 4 && xhr.status == 200) {
            var response = xhr.response.replace(new RegExp('"', 'g'), '');
            $(el).summernote('insertImage', response, file.name);
        }
    }
    return false;
}
```

First, we create a `FormData` object, append the file, post an http post request to the controller we created above. When it's done, call `insertImage`.  <a href="http://summernote.org/deep-dive/#insertimage" target="_blank">Documentation</a>

Done! :)
