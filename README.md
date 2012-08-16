MailTemplatingSystem
====================

[de] Das MailTemplatingSystem ist ein Contao basiertes Template System um E-Mails innerhalb des Backends zu erstellen und zu gestalten.
Das System kann daraus E-Mails erstellen, die mit dem MailerFramework versendet werden können.

[en] The MailTemplatingSystem is a Contao based templating system for creating mail templates within the backend.
With the ability to generate emails that can be send with the MailerFramework.

Dependencies
------------

* MailerFramework https://github.com/InfinitySoft/contao-mailer

Idee / Zusammenfassung / Entwicklerinfo
---------------------------------------

[de] Das MailTemplatingSystem wird aus Avisota extrahiert, hier soll kurz beschrieben werden, wie dies passieren soll.

Was muss extrahiert werden?

<table>
  <tr>
    <th>Avisota</th>
    <th>Ziel</th>
  </tr>
  <tr>
    <td>Avisota2/dca/tl_avisota_newsletter + Avisota2/dca/tl_avisota_newsletter_draft</td>
    <td>-> MailTemplate/dca/tl_mail_template</td>
  </tr>
  <tr>
    <td>Avisota2/dca/tl_avisota_newsletter_content + Avisota2/dca/tl_avisota_newsletter_draft_content</td>
    <td>-> MailTemplate/dca/tl_mail_template_content</td>
  </tr>
  <tr>
    <td>Avisota2/dca/tl_avisota_newsletter_theme</td>
    <td>-> MailTemplate/dca/tl_mail_template_theme oder MailTemplate/dca/tl_mail_template_layout</td>
  </tr>
  <tr>
    <td>Avisota2/templates/mail_html_* + Avisota2/templates/mail_plain_*</td>
    <td>-> MailTemplate/templates/*</td>
  </tr>
  <tr>
    <td>Avisota2/templates/nl_gallery*</td>
    <td>-> MailTemplate/templates/mt_gallery*</td>
  </tr>
  <tr>
    <td>Avisota2/templates/nle_*</td>
    <td>-> MailTemplate/templates/mte_*</td>
  </tr>
  <tr>
    <td>Avisota2/Newsletter*.php</td>
    <td>-> MailTemplate/MailTemplate*.php</td>
  </tr>
  <tr>
    <td>Avisota2/WidgetEventchooser.php</td>
    <td>-> MailTemplate/WidgetEventchooser.php</td>
  </tr>
  <tr>
    <td>Avisota2/WidgetNewschooser.php</td>
    <td>-> MailTemplate/WidgetNewschooser.php</td>
  </tr>
  <tr>
    <td>Avisota2/AvisotaNewsletterTheme.php</td>
    <td>-> MailTemplate/MailTemplateTheme.php oder MailTemplate/MailTemplateLayout</td>
  </tr>
</table>

Mail Templates soll ein zusätzliche Felder *type* bekommen.
Der Typ des Mail Templates (z.B. Avisota Vorlage, Isotope Bestellbestätigung, Isotope Anmeldebestätigung, usw.) soll anderen Entwicklern eine einfache und unabhängige Möglichkeit bieten eventuell benötigte Zusatzfelder hinzufügen können. Außerdem ist durch den Typ sofort klar, wofür das Mail Template verwendet wird. Es sollte auf jeden Fall einen generischen *default* Typ geben.

Weitere Änderungen die zu berücksichtigen sind:
* *area* soll in column *umbenannt* werden.
* Ein optionales Feld *subject* sollte dem MailTemplate hinzugefügt werden (zu dem *default* Typ), falls das benutzende System selbst keinen Subject setzt.

Neue Klassen müssen eingeführt werden (hier als einfacher Outline Prototyp):

```php
class MailTemplate
{
    /**
     * Erstellt ein MailTemplate aus der Datenbank.
     *
     * @return MailTemplate
     */
    public static function load($varId);

    /**
     * Erzeugt ein leeres MailTemplate
     */
    public function __construct();

    /**
     * Setzt das Theme das von diesem MailTemplate verwendet wird.
     *
     * @param MailTemplateTheme $objTheme
     */
    public function setTheme(MailTemplateTheme $objTheme);

    /**
     * Setzt den Inhalt dieses MailTemplates
     *
     * @param array<MailTemplateElement> $arrContent
     * @param string $column
     */
    public function setContentData(array $arrContent, $column = 'body');

    /**
     * Fügt ein weiteres Inhaltselement in der Spalte hinzu.
     *
     * @param MailTemplateElement $objContent
     * @param string $column
     */
    public function addContent(MailTemplateElement $objContent, $column = 'body');

    /**
     * Generiert den HTML Inhalt und ersetzt ggf. SimpleTokens.
     *
     * @param array $arrTokens Daten für replaceSimpleTokens.
     * @return string
     */
    public function generateHtml(array $arrTokens = null);
    // Logik und Code kann aus der AvisotaNewsletterContent extrahiert werden.

    /**
     * Generiert den Plain Inhalt und ersetzt ggf. SimpleTokens.
     *
     * @param array $arrTokens Daten für replaceSimpleTokens.
     * @return string
     */
    public function generatePlain(array $arrTokens = null);
    // Logik und Code kann aus der AvisotaNewsletterContent extrahiert werden.

    /**
     * Generiert ein Mail Objekt das mit dem Mailer versendet werden kann.
     *
     * @param array $arrTokens Daten für replaceSimpleTokens.
     * @return Mail (siehe https://github.com/InfinitySoft/contao-mailer/blob/master/src/system/modules/mailer/Mail.php)
     */
    public function generateMail(array $arrTokens = null);
}
```

Usage examples
--------------

Einfaches Beispiel, Theme und MailTemplate werden aus der Datenbank geladen.

```php
$objTheme = MailTemplateTheme::load(123);

$objMailTemplate = MailTemplate::load(456);

$objMail = $objMailTemplate->generateMail();

$objMailer = Mailer::getMailer();
$objMailer->send($objEmail, 'alex@example.com');
```

Einfaches Beispiel, nur das Theme wird aus der Datenbank geladen.

```php
$objTheme = MailTemplateTheme::load(123);

$objMailTemplate = new MailTemplate();
$objMailTemplate->setTheme($objTheme);
$objMailTemplate->addContent(new MailTemplateText(array('text' => 'Lorem ipsum')));

$objMail = $objMailTemplate->generateMail();

$objMailer = Mailer::getMailer();
$objMailer->send($objEmail, 'alex@example.com');
```
