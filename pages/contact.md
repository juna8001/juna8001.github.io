---
layout: default
title: "Contact"
permalink: /contact/
---

<style>
.contact-page {
  display: flex;
  flex-wrap: wrap;
  gap: 2rem;
}

.contact-info, .contact-form-column {
  flex: 1 1 300px; /* flexible columns, min width 300px */
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.contact-info h1 {
  margin-bottom: 1rem;
}

.contact-method {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.contact-method span {
  font-size: 1rem;
}
</style>

<div class="contact-page">
  <!-- Left column: contact info -->
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

  <!-- Right column: contact form -->
  <div class="contact-form-column">
    <h2>Send me a message</h2>
    {% include contact_form.html %}
  </div>
</div>