On form pages for the Ruby on Rails framework, a new element, `<div id='error-explanation'>` will appear when a form is submitted and an error is thrown. In order to be compliant to a user using assistive technology, the visual focus must move to the revealed content.

This can be achieved by using javascript to move the focus with the `focus` element and setting the [tabindex attribute](https://www.wufoo.com/html5/attributes/27-tabindex.html) to `tabindex=0` on the `error-explanation` element.

By default, the color contrast of the standard Rails scaffold `error-explanation` (see the `_form.html.erb` view for your model) div is usually not 508 compliant. Here's an example of how the styling can be changed to fix it:

**Note the new ID from `error-explanation` to `error_explanation`**

HTML at app/views/model_name/_form.html.erb:

```
  <% if @model_name.errors.any? %>
    <div id="error_explanation" tabindex="0">
      <h2><%= pluralize(@model_name.errors.count, "error") %> prohibited this post from being saved:</h2>
      <ul>
        <% @model_name.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

```

CSS:
```
.field_with_errors {
  padding: 2px;
  display: table;
}

#error_explanation {
  width: 450px;
  border: 2px solid red;
  padding: 7px;
  padding-bottom: 0;
  margin-bottom: 20px;
  background-color: #f0f0f0;
```
Now that the color and markup is taken care of, here's one implementation of how to go about moving the visual focus this in vanilla Javascript:

Javascript:
```
$( document ).ready(function() {
  seterrorFocus();
});

function setErrorFocus() {
  var error_explanation = document.getElementById('error_explanation');
  if (error_explanation != null) {
    error_explanation.focus();
    // There's nothing special about the 'focus' class. It's just a class
    // To test for executing the above code to add the focus for the feature test
    error_explanation.classList.add('focus');
  };
}

```

Feature test at spec/model_name/model_name_create_spec.rb:

```
RSpec.describe 'model_name creation page', js: true do
  context 'Accessibility Related' do
    it 'should bring visual focus to first missing field element', js: true do
        click_button 'Create Model_name'
        // Choose whatever the error page brings to
        expect(page.current_path).to eq model_name_page_path
        // Can check for the full element to assure the tabindex is 0
        expect(page).to have_selector(
          :css,
          "div[id='error_explanation']" \
          "[tabindex='0']"
        )
        // We added the class with javascript to assure that the full code executed
        page.find("#error_explanation")[:class].include?("focus")
      end
    end
  end
end

