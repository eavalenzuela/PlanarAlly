<template>
    <form @submit.prevent>
        <div class="spanrow header">General</div>
        <div class="row">
            <label for="username">Username:</label>
            <div>
                <input
                    type="text"
                    id="username"
                    ref="username"
                    :value="$store.state.core.username"
                    autocomplete="username"
                    readonly
                />
            </div>
        </div>
        <div class="row">
            <label for="email">Email:</label>
            <div>
                <input
                    type="email"
                    id="email"
                    :placeholder="$store.state.core.email === undefined ? 'no email set' : ''"
                    :value="$store.state.core.email"
                    autocomplete="email"
                    @change="updateEmail"
                />
            </div>
        </div>
        <div class="spanrow header">Danger Zone</div>
        <div class="row" v-if="showPasswordFields">
            <label for="password-reset">New Password:</label>
            <div>
                <input ref="passwordResetField" type="password" id="password-reset" autocomplete="new-password" />
            </div>
        </div>
        <div class="row" v-if="showPasswordFields">
            <label for="password-repeat">Repeat password:</label>
            <div>
                <input ref="passwordRepeatField" type="password" id="password-repeat" autocomplete="new-password" />
            </div>
        </div>
        <div class="spanrow" v-show="errorMessage" style="display:flex;justify-content:center;">
            <span v-show="errorMessage" class="danger" style="font-weight:bold;">{{ errorMessage }}</span>
        </div>
        <div class="row">
            <div>
                <button class="danger" v-if="showPasswordFields" @click="hidePasswordChange">
                    Cancel
                </button>
            </div>
            <div>
                <button class="danger" @click="changePassword" ref="changePasswordButton">Change password</button>
            </div>
        </div>
        <div class="row">
            <div style="grid-column-start: value">
                <button class="danger" @click="deleteAccount">Delete account</button>
            </div>
        </div>
        <ConfirmDialog ref="confirm">
            This action is irrevocable!
        </ConfirmDialog>
    </form>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";

import ConfirmDialog from "@/core/components/modals/confirm.vue";

import { coreStore } from "../core/store";
import { postFetch } from "../core/utils";

@Component({
    components: {
        ConfirmDialog,
    },
})
export default class AccountSettings extends Vue {
    $refs!: {
        confirm: InstanceType<typeof ConfirmDialog>;
        changePasswordButton: HTMLButtonElement;
        passwordResetField: HTMLInputElement;
        passwordRepeatField: HTMLInputElement;
    };

    showPasswordFields = false;
    errorMessage = "";

    async updateEmail(event: { target?: HTMLInputElement }): Promise<void> {
        if (event.target?.checkValidity() && event.target.value !== this.$store.state.core.email) {
            const result = await postFetch("/api/users/email", {
                email: event.target.value,
            });
            if (result.ok) {
                coreStore.setEmail(event.target.value);
                // todo: show some kind of notification to notify of success
            } else {
                event.target.value = coreStore.email ?? "";
            }
        }
    }

    async changePassword(): Promise<void> {
        if (this.showPasswordFields) {
            if (this.$refs.passwordResetField.value === "") {
                this.errorMessage = "No new password given!";
                return;
            }
            if (this.$refs.passwordRepeatField.value !== this.$refs.passwordResetField.value) {
                this.errorMessage = "The password fields don't match!";
                return;
            }
            const response = await postFetch("/api/users/password", { password: this.$refs.passwordResetField.value });
            if (response.ok) {
                this.hidePasswordChange();
            } else {
                this.errorMessage = response.statusText ?? "Something went wrong during the server request";
            }
        } else {
            this.showPasswordFields = true;
            this.$refs.changePasswordButton.textContent = "Confirm";
            this.$nextTick(() => this.$refs.passwordResetField.focus());
        }
    }

    hidePasswordChange(): void {
        this.showPasswordFields = false;
        this.errorMessage = "";
        this.$refs.changePasswordButton.textContent = "Change password";
    }

    async deleteAccount(): Promise<void> {
        const result = await this.$refs.confirm.open("Are you sure you wish to remove your account?");
        if (result) {
            const response = await postFetch("/api/users/delete");
            if (response.ok) {
                coreStore.setAuthenticated(false);
                this.$router.push("/");
            } else {
                this.errorMessage = "Something went wrong with the delete request.";
            }
        }
    }
}
</script>

<style scoped>
* {
    -ms-box-sizing: border-box;
    -moz-box-sizing: border-box;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    border: 0;
}

:invalid {
    background-color: rgba(255, 0, 0, 0.1);
}

main {
    margin: auto;
    display: grid;
    grid-template-areas:
        "title title"
        "nav   main";
    /* "nav   main  main"; */
    grid-template-rows: 60px 1fr;
    grid-template-columns: 100px 1fr;
}

#title {
    grid-area: title;
    background: #ff7052;
    padding: 15px;
    color: #fff;
    font-size: 20px;
}

nav {
    grid-area: nav;
    background-color: lightblue;
}

nav > div {
    padding: 0.5em;
}

.main {
    background-color: white;
    grid-area: main;
    padding-left: 1em;
    padding-right: 1em;
    display: grid;
    grid-template-columns: [setting] 1fr [value] 1fr [end];
    /* align-items: center; */
    align-content: start;
    min-height: 10em;
}

.row {
    display: contents;
}

.row > *,
.panel > *:not(.row) {
    display: flex;
    /* justify-content: center; */
    align-items: center;
    padding: 0.5em;
}

.row:first-of-type > * {
    margin-top: 0.5em;
}

.row:last-of-type > * {
    margin-bottom: 0.5em;
}

.row:hover > * {
    cursor: pointer;
    text-shadow: 0px 0px 1px black;
}

.smallrow > * {
    padding: 0.2em;
}

.header {
    line-height: 0.1em;
    margin: 20px 0 15px;
    font-style: italic;
}
.header:after {
    position: relative;
    left: 5px;
    width: 100%;
    border-bottom: 1px solid #000;
    content: "";
}

.spanrow {
    grid-column: 1 / end;
}

.danger {
    color: #ff7052;
}

.danger:hover {
    text-shadow: 0px 0px 1px #ff7052;
    cursor: pointer;
}

input[type="checkbox"] {
    width: 16px;
    height: 23px;
    margin: 0;
    white-space: nowrap;
    display: inline-block;
}

input[type="number"],
input[type="text"],
input[type="email"],
input[type="password"] {
    width: 100%;
    padding: 5px;
}
button {
    padding: 6px 12px;
    border: 1px solid lightgray;
    border-radius: 0.25em;
    background-color: rgb(235, 235, 228);
}
</style>
