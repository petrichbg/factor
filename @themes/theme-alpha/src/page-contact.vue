<template>
  <div class="page-contact">
    <el-hero
      :headline="setting('contact.headline')"
      :subheadline="setting('contact.subheadline')"
      :image="setting('contact.heroImage')"
    >
      <template v-slot:hero-content>
        <div v-formatted-text="setting('contact.content')" class="content entry-content" />

        <factor-form
          ref="form"
          class="contact-form"
          data-test="contact-form"
          :class="formStatus"
          @submit="send()"
        >
          <div v-if="sent" class="confirm" data-test="confirm">
            <div class="title">Got it!</div>
            <div
              class="description"
            >We’ll get back to you as soon as possible at the email you provided.</div>
          </div>
          <div v-else class="inputs">
            <factor-input-wrap
              v-model="form.name"
              format="vertical"
              data-test="form-name"
              input="factor-input-text"
              :placeholder="setting('contact.form.namePlaceholder')"
              required
            />
            <factor-input-wrap
              v-model="form.email"
              format="vertical"
              data-test="form-email"
              input="factor-input-email"
              :placeholder="setting('contact.form.emailPlaceholder')"
              required
            />
            <factor-input-wrap
              v-model="form.message"
              format="vertical"
              input="factor-input-textarea"
              :placeholder="setting('contact.form.messagePlaceholder')"
              required
              data-test="form-message"
            />

            <factor-input-submit
              btn="default"
              size="large"
              :loading="sending"
              data-test="form-submit"
            >
              {{ setting("contact.form.buttonText") }}
              <factor-icon icon="fas fa-arrow-right" />
            </factor-input-submit>
          </div>
        </factor-form>
      </template>
    </el-hero>
  </div>
</template>

<script lang="ts">
import { factorForm, factorInputSubmit, factorInputWrap, factorIcon } from "@factor/ui"
import { setting } from "@factor/api"
import Vue from "vue"
export default Vue.extend({
  components: {
    factorForm,
    factorInputSubmit,
    factorInputWrap,
    factorIcon,
    "el-hero": () => import("./el/hero.vue")
  },
  data() {
    return {
      loading: true,
      sending: false,
      form: {},
      sent: false,
      formStatus: "unchecked"
    }
  },
  metaInfo() {
    return {
      title: setting("contact.metatags.title"),
      description: setting("contact.metatags.description")
    }
  },
  mounted() {
    this.$watch(
      "form",
      function(this: any) {
        const v = this.$refs.form.$el.checkValidity()

        this.formStatus = v ? "valid" : "invalid"
      },
      { deep: true }
    )
  },
  // pageTemplate() {
  //   return {
  //     name: "Contact Page",
  //     inputs: [
  //       {
  //         input: "text",
  //         label: "Heading",
  //         key: "pageHeading"
  //       },
  //       {
  //         input: "image-upload",
  //         label: "Image",
  //         key: "heroImage"
  //       }
  //     ]
  //   }
  // },
  methods: {
    setting,
    async send(this: any) {
      this.sending = true
      const { email } = this.form
      // this.$email.send({
      //   to: ["raylopezaleman@gmail.com"],
      //   subject: `Contact Form: ${name} ${email}`,
      //   message: `A form was submitted by ${name}.`,
      //   meta: this.form
      // })

      // if (!name || !email || !message) {
      //   this.$notify.error("Please enter your contact information into the form.")
      //   return
      // }

      try {
        const _p = []

        _p.push(
          this.$email.send({
            to: email,
            subject: `Got your message.`,
            message: `This is to confirm we've recieved a form you submitted. We'll take a look and be in touch as soon as possible.`,
            title: "Contact",
            table: this.form
          })
        )

        _p.push(
          this.$sheets.saveForm({
            ...this.form,
            source: "/contact",
            category: "General",
            type: "Contact"
          })
        )

        await Promise.all(_p)

        this.sent = true
      } catch (error) {
        this.$notify.error("There was an issue sending your form.")
      }

      this.sending = false
    }
  }
})
</script>

<style lang="less">
.page-contact {
  .mast {
    padding: 0 2em;
    line-height: 1.2;
    max-width: 1000px;
    margin: 0 auto;
  }

  .contact-form {
    margin-top: 40px;
    input[type="text"],
    input[type="email"] {
      width: 100%;
    }
    input[type="text"],
    input[type="email"],
    textarea.standard-textarea {
      background: var(--color-white, #ffffff);
      border: 1px solid rgba(48, 48, 48, 0.1);
      border-radius: 4px;
      &:focus {
        outline: var(--color-primary, #9afecb) auto 5px;
      }
    }
    .form-submit {
      margin: 1em 0;
    }
  }
  .confirm {
    padding: 8em 2em;
    text-align: center;
    .title {
      font-size: 1.7em;
    }

    .description {
      opacity: 0.5;
    }
    .actions {
      margin-top: 1em;
    }
  }
}
</style>
