# Possibility to access an arbitrary file

<table class='noborder'>
    <colgroup>
      <col/>
      <col/>
    </colgroup>
    <tbody>
      <tr>
        <td rowspan="2"><img src="../../../img/defekt_srednij.png"/></td>
        <td>Severity:<strong> MEDIUM</strong></td>
      </tr>
      <tr>
        <td>Detection method:<strong> IAST</strong></td>
      </tr>
    </tbody>
</table>
## Description

This vulnerability allows to get access to arbitrary application files.

The vulnerability is present in applications that do not properly validate the input Uri-parameter and use it to access file handling methods. A malicious application can specifically generate a Uri, pass it in one way or another to a vulnerable application, and gain access to an arbitrary file.

An example of vulnerable code:

    private void takeSomeSharedFile() {
    Intent interceptableIntent = new Intent("vuln.app.pkg.INTERCAPTABLE");
    startActivityForResult(interceptableIntent, 1001);
    }
    
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if(resultCode == -1 && data != null) {
        if(requestCode == 1001) {
        copyToCache(data.getData());
        }
    }
    }
    
    public static File copyToCache(Uri uri) {
    try {
        File out = new File(getExternalCacheDir(), "" + System.currentTimeMillis());
        InputStream i = getContentResolver().openInputStream(uri);
        OutputStream o = new FileOutputStream(out);
        IOUtils.copy(i, o);
        i.close();
        o.close();
        return out;
    }
    catch (Exception e) {
        return null;
    }
    }

A malicious application can use such code:

    Intent intent = new Intent();
    File privFile = new File("/data/data/vuln.app.pkg/databases/main.db");
    intent.setData(Uri.fromFile(privFile));
    setResult(RESULT_OK, intent);
    finish();

As a result, the vulnerable application will copy the ***main.db*** file to the `/storage/emulated/0/Android/data/vuln.app.pkg/1650464271` file, which can be accessed by the malicious application.

## Recommendations

It is necessary to validate the canonical path of a file just before calling file operation methods on that file:

    File file = new File(uri);
    if (!file.getCanonicalPath().startsWith(sdcardDir)) {
    throw new IllegalArgumentException();
    }

.