<template>
    <div class="govuk-width-container">
        <main class="govuk-main-wrapper">
            <div class="govuk-grid-column-full">
                <h2 class="govuk-heading-l">
                    Máte nápad ako zlepšiť elektronické služby ?
                </h2>
                <p>
                    Poďme do toho spolu a ukážme, kde je priestor na zlepšenie.
                </p>
            </div>
            <div class="govuk-grid-column-two-thirds">
                <form id="submit-form" action="/" method="post">
                    <fieldset class="govuk-fieldset">
                        <div class="govuk-form-group">
                            <label class="govuk-label" for="govuk-input-summary"
                                >Akú situáciu riešite? Krátky popis.</label
                            >
                            <input
                                id="govuk-input-summary"
                                v-model="form.summary"
                                class="govuk-input"
                                :class="{ 'error-input': !valid.summary }"
                                name="govuk-input-summary"
                                type="text"
                                @blur="
                                    valid.summary = validInput(
                                        form.summary,
                                        'text'
                                    )
                                "
                            />
                            <span class="govuk-error-message">
                                <span class="govuk-visually-hidden"
                                    >Error:</span
                                >
                                Toto pole je povinné.
                            </span>
                        </div>

                        <div class="govuk-form-group">
                            <label
                                class="govuk-label"
                                for="govuk-textarea-description"
                                >Text podnetu</label
                            >
                            <textarea
                                id="govuk-textarea-description"
                                v-model="form.description"
                                class="govuk-textarea"
                                :class="{ 'error-input': !valid.description }"
                                name="govuk-textarea-description"
                                rows="5"
                                @blur="
                                    valid.description = validInput(
                                        form.description,
                                        'text'
                                    )
                                "
                            ></textarea>
                            <span class="govuk-error-message">
                                <span class="govuk-visually-hidden"
                                    >Error:</span
                                >
                                Toto pole je povinné.
                            </span>
                        </div>

                        <div class="govuk-form-group">
                            <label class="govuk-label" for="select-category">
                                Kategória
                                <em>(nepovinné)</em>
                            </label>
                            <nd-select
                                v-model="form.categories.persona"
                                :options="categories.persona"
                            ></nd-select>
                            <nd-select
                                v-if="subCategories.length > 0"
                                v-model="form.categories.field"
                                :options="subCategories"
                            ></nd-select>
                        </div>

                        <div class="govuk-form-group">
                            <label class="govuk-label" for="file-upload">
                                Príloha
                                <em>(nepovinné)</em>
                            </label>
                            <nd-files @file="form.files = $event"></nd-files>
                        </div>

                        <div class="govuk-form-group">
                            <label class="govuk-label" for="govuk-input-name">
                                Meno
                                <em>(nepovinné)</em>
                            </label>
                            <input
                                id="govuk-input-name"
                                v-model="form.name"
                                class="govuk-input govuk-!-width-two-thirds"
                                name="govuk-input-name"
                                type="text"
                            />
                        </div>

                        <div class="govuk-form-group">
                            <label class="govuk-label" for="govuk-input-email">
                                E-mail
                                <em>(nepovinné)</em>
                            </label>
                            <input
                                id="govuk-input-email"
                                v-model="form.email"
                                class="govuk-input govuk-!-width-two-thirds"
                                :class="{ 'error-input': !valid.email }"
                                name="govuk-input-email"
                                type="text"
                                @blur="
                                    valid.email =
                                        form.email === '' ||
                                        validInput(form.email, 'email')
                                "
                            />
                            <span class="govuk-error-message">
                                <span class="govuk-visually-hidden"
                                    >Error:</span
                                >
                                Email je v nesprávnej forme.
                            </span>
                        </div>

                        <button
                            class="govuk-button"
                            :disabled="!gdpr || !isValidForm"
                            @click.prevent="createIssue"
                        >
                            Vytvoriť
                        </button>
                        <div class="govuk-checkboxes suhlas">
                            <div class="govuk-checkboxes__item">
                                <input
                                    v-model="gdpr"
                                    class="govuk-checkboxes__input"
                                    type="checkbox"
                                />
                                <label
                                    class="govuk-label govuk-checkboxes__label"
                                >
                                    Súhlasím so spracovaním osobných údajov v
                                    súlade s nariadením
                                    <a
                                        href="https://slovensko.digital/ochrana-osobnych-udajov"
                                        target="_blank"
                                        >GDPR o ochrane osobných údajov</a
                                    >.
                                </label>
                            </div>
                        </div>
                    </fieldset>
                </form>
            </div>
        </main>
    </div>
</template>

<script>
import persona from "../jira_personas";
export default {
    data() {
        return {
            token: "",
            gdpr: false,
            hasSent: false,
            hasOptions: false,
            form: {
                categories: {
                    persona: "Žiadne",
                    field: null,
                },
                summary: "",
                description: "",
                files: [],
                name: "",
                email: "",
            },
            valid: {
                // True at the begining for the scss class
                description: true,
                summary: true,
                email: true,
            },
            categories: {
                persona,
            },
        };
    },
    computed: {
        isValidForm: function () {
            let validity = false;
            // This disables the button if email is not it the right form
            let email = true;
            if (this.form.email !== "") {
                email = this.validInput(this.form.email, "email");
            }

            if (
                this.validInput(this.form.summary, "text") &&
                this.validInput(this.form.description, "text") &&
                email
            ) {
                validity = true;
            }
            return validity;
        },
        subCategories: {
            get: function () {
                // There have to be options
                let options = [];
                let persona = this.form.categories.persona;
                if (persona && persona !== "Žiadne") {
                    let index = this.categories.persona.findIndex(
                        (category) => category.value == persona
                    );
                    if (this.categories.persona[index].children) {
                        options = options.concat(
                            this.categories.persona[index].children
                        );
                    }
                }
                return options;
            },
        },
    },
    methods: {
        async createIssue() {
            // (optional) Wait until recaptcha has been loaded.
            await this.$recaptchaLoaded();

            // Execute reCAPTCHA with action "login".
            const token = await this.$recaptcha("login");

            // Do stuff with the received token.
            this.token = token;
            this.submitForm();
        },
        validInput: function (value, type) {
            let answer = false;
            let regEmail = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;

            switch (type) {
                case "email":
                    answer = regEmail.test(value);
                    break;
                case "text":
                    if (value !== "") {
                        answer = true;
                    }
                    break;
                default:
                    break;
            }

            return answer;
        },
        logout: async function () {
            const logoutURL =
                "https://lepsiesluzby.sk/jira/rest/auth/1/session";
            try {
                await this.axios.delete(logoutURL);
            } catch (error) {
                console.log(error);
            }
        },
        postData: async function (inputData) {
            const postURL = "https://lepsiesluzby.sk/jira/rest/api/2/issue";
            const config = {
                headers: {
                    "Content-Type": "application/json",
                    Accept: "application/json",
                    "recaptcha-token": this.token,
                },
            };
            try {
                const post = await this.axios.post(postURL, inputData, config);
                if (this.form.files && post) {
                    try {
                        const attachmentURL =
                            post.data.self + "/" + "attachments";
                        const attachmentConfig = {
                            headers: {
                                "content-type": "multipart/form-data",
                                "X-Atlassian-Token": "nocheck",
                            },
                        };
                        const formFiles = new FormData();

                        this.form.files.forEach((file) => {
                            formFiles.append("file", file);
                        });
                        await this.axios.post(
                            attachmentURL,
                            formFiles,
                            attachmentConfig
                        );
                        this.logout();
                        this.$router.push("success");
                    } catch (error) {
                        console.log(error);
                    }
                } else {
                    this.logout();
                    this.$router.push("success");
                }
            } catch (error) {
                this.$router.push({ name: "error", params: { data: error } });
                console.log(error);
            }
        },
        submitForm: function () {
            // TODO: Add validation, change customfields to be more general?
            let data = this.form;
            let cfStr = "customfield_";

            let categoryStr = cfStr + "10204"; // data.categories.persona;
            const output = {
                fields: {
                    project: {
                        key: "SDM",
                    },
                    issuetype: {
                        name: "Bug",
                    },
                    summary: data.summary,
                    description: data.description,
                    components: [
                        {
                            name: "e-services",
                        },
                    ],
                    customfield_10200: data.email.toLowerCase(),
                    customfield_10116: data.name,
                },
            };
            if (
                data.categories.persona &&
                data.categories.persona !== "Žiadne"
            ) {
                if (data.categories.field) {
                    output.fields[categoryStr] = {
                        value: data.categories.persona,
                        child: {
                            value: data.categories.field,
                        },
                    };
                }
            }

            this.postData(output);
            this.hasSent = true;
        },
    },
};
</script>

<style lang="scss">
.suhlas {
    margin: 0 0 1rem 0;
    & label {
        padding: 0 0 0 1rem;
        font-size: 1rem;
        width: 75%;
    }
}
.error-input {
    outline: 3px solid red;
    & + .govuk-error-message {
        visibility: visible;
    }
}
.govuk-error-message {
    visibility: hidden;
    margin: 0.75rem 0;
}
#govuk-textarea-description {
    margin-bottom: 0;
}
</style>
