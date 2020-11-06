---
description: 'Version 6: HTML and CSS Continued: HTML5'
---

# HTML Forms and iFrames

## Notes

### Forms

* The `<form>` element wraps all the input elements that will collect our users' information inside of it. The form element has two important attributes: `action` and `method`. The `action` attribute will specify where the user information is sent to. This is typically the URL of a remote server. The `method` attribute dictates the manner in which we submit our information. The most common HTTP methods are `GET`, `POST`, `PATCH`, `PUT`, and `DELETE`.

#### **`GET`**

* Example code for making a [`GET` request](html-forms-and-iframes.md#http-requests)
* When the user clicks the submit button of our form, all their responses are captured and labeled using the name attributes on each element. Then they are sent to the location listed in the action attribute. The request uses the method attribute set as `GET`. This causes the information to be sent as a Query String included into the URL.
  * By looking at the headers of our request in the Developer Tools &gt; Network tab, we can see the request type listed as `GET`, and our user's response is located in the Query String Parameters.
* An HTTP `GET` request is used when we want to get back particular content from the server and we want to pass it some options to refine the search of what we are getting back from the server. Since the content of our request is visible in the URL string at the top of the browser window, the is not really ideal for passwords.

#### **`POST`**

* Example code for making a [`POST` request](html-forms-and-iframes.md#http-requests)
* When the user clicks the submit button of our form, all their responses are captured and labeled using the name attributes on each element. Then they are sent to the location listed in the action attribute. The request uses the method attribute set as `POST`. This causes the information to be sent as Form Data.
  * By looking at the headers of our request in the Developer Tools &gt; Network tab, we can see the request type listed as `POST`, and our user's response is nested under Form Data.
* HTTP `POST` request is ideally used when we wish to post content to the server. This is the more appropriate of the two methods for sending something like a password as the content is sent within Form Data and not directly visible in a query string in the URL as they are with `GET` requests.

### Form Elements

#### **Text Inputs**

* Setting the input element with a `type="text"` gives our users a place to type a single line of text content. The `placeholder` attribute puts some dummy text into the element that will be replaced as the user starts typing. The `name` attribute gives our input value a label. This makes it possible to grab its value at the other end on the server by calling up the value by giving its name \(key\).
* Setting the input element with a `type="password"` displays dots as the user types characters instead of the actual characters.
* Setting the input element with a `type="tel"` will bring up the numeric keypad on supported mobile devices.

#### **Hidden Inputs**

* Setting the input element with a `type="hidden"` will send the inserted `value` attributes data along with the form submission, but won't make the data visible in the browser window.

#### **Submit Buttons**

* Setting the `input` element with a `type="submit"` will create a submission button that will submit the form upon clicking it. The `value` attribute holds the text that will appear on the button.
* You can also use a `button` element but the text that will appear on the button goes between the opening and closing tags.

#### **Radio Inputs**

* Radio inputs are useful when you want to have several possible values but wish to limit the user to choosing one or another. To accomplish this, we must set different `value` attributes for each radio button, but they must share the same `name` attribute.

#### **Checkboxes**

* Checkboxes are ideal if we wish to allow our users to choose one or more values. They are structured the same as radio inputs only with a `type="checkbox"` instead.

#### **Select Menus**

* Select menus create a nice drop menu with multiple selectable options. Here the user must also choose a single option, so this could be a replace for a set of radio buttons. Here we must assign a text label between the opening and closing option elements as well as a `value` attribute that will hold the choice the user selects that will be passed along during form submission.

#### **Textarea**

* Textarea elements are useful if we want our users to be able to insert multiple lines of text.

#### **iFrames**

* iFrame elements allow us to link to other HTML content from within a frame window on our pages. The `src` attribute points to the location of other HTML content elsewhere and displays it within the current page.

#### **Labels**

* Use a `<label>` element to create a text label associated with the form input. By using the `for` attribute set on the label to match the value of the `id` attribute in the input, this ties the label to the input so that if a user click in the text of the label, it will select the input next to it.

#### **Required**

* The `required` attribute helps us alert the user when they try to submit the form without filling out a required input field.

## Code Examples

### HTTP Requests

{% tabs %}
{% tab title="GET Request" %}
```markup
<form action="process-user.php" method="get">
  <input type="text" name="full-name">
  <input type="password" name="password">
  <input type="submit" value="submit">
</form>

#URL for a GET request
localhost:9292/form-example/process-user.php?full-name=Bob+Barkley&password=fishyfish
```
{% endtab %}

{% tab title="POST Request" %}
```markup
<form action="process-user.php" method="post">
  <input type="text" name="full-name">
  <input type="password" name="password">
  <input type="submit" value="submit">
</form>
```
{% endtab %}
{% endtabs %}

### Form Elements

{% tabs %}
{% tab title="Text Input" %}
```markup
<input type="text" name="username" placeholder="username">
```
{% endtab %}

{% tab title="Hidden Input" %}
```markup
<input type="hidden" name="id" value="UID-99298356982">
```
{% endtab %}

{% tab title="Submit Button" %}
```markup
<!-- Using an input element with type submit -->
<input type="submit" value="submit">

<!-- Using a button element -->
<button type="submit">Submit</button>
```
{% endtab %}

{% tab title="Radio Input" %}
```markup
<input type="radio" name="gender" value="male"> Male
<input type="radio" name="gender" value="female"> Female
```
{% endtab %}

{% tab title="Select Menu" %}
```markup
<select name="size">
  <option value="small" selected>small</option>
  <option value="medium">medium</option>
  <option value="large">large</option>
</select>
```
{% endtab %}

{% tab title="Textarea" %}
```markup
<textarea name="message"></textarea>
```
{% endtab %}
{% endtabs %}

