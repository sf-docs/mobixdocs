# Including sensitive information into the GET request parameters

<table class='noborder'>
    <colgroup>
      <col/>
      <col/>
    </colgroup>
    <tbody>
      <tr>
        <td rowspan="2"><img src="../../../img/defekt_vysokij.png"/></td>
        <td>Severity:<strong> HIGH</strong></td>
      </tr>
      <tr>
        <td>Detection method:<strong> DAST, NETWORKING</strong></td>
      </tr>
    </tbody>
</table>
## Description

Some applications use the GET method for passing confidential information (such as passwords, session identifiers etc.) which is displayed in a URL as parameters. Confidential information from URLs might be registered in a variety of places including:

* Web servers;
* Any forward or reverse proxy servers between two endpoints;
* Referrer headers;
* Web server logs;
* Browser history;
* Browser cache.

URLs can also be shown on a display, added to bookmarks, or sent to a user via email. They can be passed to third parties in a Referrer header when a user follows external links. Placing confidential information into URL adds up to a risk of a security breach.

Vulnerabilities that lead to disclosure of a user's confidential information can also lead to compromises that are very difficult to research. Even if the application itself handles non-sensitive information only, disclosure of information such as passwords put users at risk if they re-use this password in a different application.

## Recommendations

All requests containing sensitive information must use the POST-method; the confidential information must be put into the request body to ensure that the information doesn't get into web server logs or other places accessible by many users. If it's not possible to change the GET-method to POST, additional encryption or hashing of confidential information can be used to prevent data compromisation.

## Links

1. [https://owasp.org/www-community/vulnerabilities/Information\_exposure\_through\_query\_strings\_in\_url#:~:text=Description,any%20other%20potentially%20sensitive%20data.](https://owasp.org/www-community/vulnerabilities/Information_exposure_through_query_strings_in_url#:~:text=Description,any%20other%20potentially%20sensitive%20data.)

2. [https://cwe.mitre.org/data/definitions/598.html](https://cwe.mitre.org/data/definitions/598.html)

3. [https://portswigger.net/kb/issues/00400300\_password-submitted-using-get-method](https://portswigger.net/kb/issues/00400300_password-submitted-using-get-method)