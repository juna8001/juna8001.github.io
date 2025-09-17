---
layout: default
title: "Contact"
permalink: /contact/
---

<div class="contact-page">
  <div class="contact-info">
    <h1>CONTACT ME:</h1>

    <div class="contact-method">
      {% include buttons/phone_button.html %}
      <span>Call me at +48 123 456 789</span>
    </div>

    <div class="contact-method">
      {% include buttons/email_button.html %}
      <span>Email me at example@example.com</span>
    </div>

    <div class="contact-method">
      {% include buttons/github_button.html %}
      <span>Check my GitHub projects</span>
    </div>

    <div class="contact-method">
      {% include buttons/linkedin_button.html %}
      <span>Connect with me on LinkedIn</span>
    </div>
  </div>

  <div class="contact-form-column">
    <h1>OR MESSAGE DIRECTLY</h1>
    {% include contact_form.html %}
  </div>
</div>