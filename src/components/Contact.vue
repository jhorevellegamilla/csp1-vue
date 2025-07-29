<template>
  <div class="container-fluid text-center py-5" id="contact">
    <h1>Contacts</h1>
    <div class="row justify-content-center" id="map">
      <div class="col-md-4">
        <iframe
          src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d30429.266546192266!2d120.36817416822508!3d17.571447052366086!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x338e656308bfb267%3A0xc81fb703f1e000f3!2sCalle%20Crisologo!5e0!3m2!1sen!2sph!4v1750147777810!5m2!1sen!2sph"
          width="100%" height="400" style="border:0;" allowfullscreen="" loading="lazy"
          referrerpolicy="no-referrer-when-downgrade">
        </iframe>
      </div>

      <div class="col-md-4 d-flex flex-column align-items-start">
        <form @submit.prevent="submitForm" class="w-100">
          <div class="mb-3">
            <label for="fullName" class="form-label">Full Name</label>
            <input type="text" v-model="name" class="form-control" id="fullName" name="fullName" placeholder="First Name M.I Last Name" required>
          </div>

          <div class="mb-3">
            <label for="email" class="form-label">Email Address</label>
            <input type="email" v-model="email" class="form-control" id="email" name="email" placeholder="Email" required>
          </div>

          <div class="mb-3">
            <label for="message" class="form-label">Message</label>
            <textarea v-model="message" class="form-control" id="message" name="message" rows="3" placeholder="Message" required></textarea>
          </div>

          <div class="d-flex justify-content-between align-items-center mt-2">
            <div class="d-flex gap-3">
              <a href="https://www.linkedin.com/" target="_blank">
                <img src="https://cdn-icons-png.flaticon.com/512/174/174857.png" alt="LinkedIn" width="30">
              </a>
              <a href="https://github.com/" target="_blank">
                <img src="https://cdn-icons-png.flaticon.com/512/25/25231.png" alt="GitHub" width="30">
              </a>
            </div>
            <button type="submit" class="btn btn-primary" :disabled="isLoading">
              {{ isLoading ? "Sending..." : "Submit" }}
            </button>
          </div>

          <div class="d-flex justify-content-end mt-2">
          	<div ref="recaptchaContainer"></div>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>

	import { ref, onMounted, onBeforeUnmount } from 'vue';
	import { Notyf } from 'notyf';
	import 'notyf/notyf.min.css';

	const notyf = new Notyf();
	const WEB3FORMS_ACCESS_KEY = "591c13e5-1265-4846-b417-ec0a081347f7"

	const subject = "New message from portfolio contact form";

	const name = ref("");
	const email = ref("");
	const message = ref("");

	const isLoading = ref(false);

	// The submitForm() handler function sends the form data to web3forms and displays success or failure notifications toast.
	const submitForm = async () => {

			if(!recaptchaToken.value){
				notyf.error('Please verify that you are not a robot.');
				return;
			}

	    // Set the loading state to true.
	    isLoading.value = true;
	    try{
	        // Send the form data to web3forms using fetch() API.
	        // The fetch() API is used to send HTTP requests to a server.
	        // Sends a POST request to the https://api.web3forms.com/submit endpoint.
	        // The request body contains the form data and the web3forms access key.
	        const response = await fetch("https://api.web3forms.com/submit", {
	            method: "POST",
	            headers: {
	                "Content-Type": "application/json",
	                Accept: "application/json",
	            },
	            body: JSON.stringify({
	                access_key: WEB3FORMS_ACCESS_KEY,
	                subject: subject,
	                name: name.value,
	                email: email.value,
	                message: message.value,
	            }),
	        });
	        const result = await response.json();

	        // Check if the form submission was successful.
	        // If successful, display success notification toast.
	        if (result.success) {
	            console.log(result);
	            // Set the loading state to false.
	            isLoading.value = false;
	            notyf.success("Message Sent!");
	        }
	    } catch (error) {

	        // If not successful, display failure notification toast.
	        console.log(error);
	        // Set the loading state to false.
	        isLoading.value = false;
	        notyf.error("Failed to send message.");
	    } finally {
	    		resetRecaptcha();
	    }
	}

	/** 
	 * 
	 * reCaptcha Integration 
	 * 
	 */

	const SITE_KEY = '6LddtZIrAAAAAPqxiCP0Ll-hfD4WOJjK_snbuwsR';  // Replace with your site key

	const recaptchaContainer = ref(null);
	const recaptchaWidgetId = ref(null);
	const recaptchaToken = ref('');

	// Callback called by reCAPTCHA when successful
	function onRecaptchaSuccess(token) {
	  recaptchaToken.value = token;
	}

	// Callback when expired
	function onRecaptchaExpired() {
	  recaptchaToken.value = '';
	}

	// Function to render the reCAPTCHA widget
	function renderRecaptcha() {
	  if (!window.grecaptcha) {
	    console.error('reCAPTCHA not loaded');
	    return;
	  }

	  recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
	    sitekey: SITE_KEY,
	    size: 'normal', // or 'compact'
	    callback: onRecaptchaSuccess,
	    'expired-callback': onRecaptchaExpired,
	  });
	}

	// Function to reset reCAPTCHA 
	function resetRecaptcha() {
	  if (recaptchaWidgetId.value !== null) {
	    window.grecaptcha.reset(recaptchaWidgetId.value);
	    recaptchaToken.value = '';
	  }
	}

	let interval;

	onMounted(() => {
	  // This code waits for the Google reCAPTCHA library to load, then renders the reCAPTCHA widget using onMounted hook. 
	  // The widget is rendered with grecaptcha.render(), which requires a sitekey. 
	  // Callback functions handle success and expiration events. 
	  // reCAPTCHA is reset upon form submission to clear the token.
	  const interval = setInterval(() => {
	    if (window.grecaptcha && window.grecaptcha.render) {
	      renderRecaptcha();
	      clearInterval(interval);
	    }
	  }, 100);

	});

	  onBeforeUnmount(() => {
	    clearInterval(interval);
	  });
	
</script>