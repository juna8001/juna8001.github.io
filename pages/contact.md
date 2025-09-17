---
layout: default
title: "Contact"
permalink: /contact/
---

<div class="contact-page">
  <div class="contact-info">
    <h2>CONTACT ME:</h2>

    <div class="contact-method">
      {% include buttons/phone_button.html %}
      <span>Call me at {{ site.data.contact.phone }}</span>
    </div>

    <div class="contact-method">
      {% include buttons/email_button.html %}
      <span>Email me at {{ site.data.contact.email }}</span>
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
    <h2>OR MESSAGE DIRECTLY:</h2>
    {% include contact_form.html %}
  </div>
</div>