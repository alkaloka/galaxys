<template>
    <div class="overflow-auto m-3">
        <ChangePassword
            v-if="hasToken"
            :expired-user="$route.query.expired_user"
            :message-text="$route.query.message"
            :message-variant="$route.query.status"
            :token="$route.query.token" />
        <LoginIndex
            v-else
            :allow-user-creation="config.allow_user_creation"
            :enable-oidc="config.enable_oidc"
            :mailing-join-addr="config.mailing_join_addr"
            :prefer-custos-login="config.prefer_custos_login"
            :redirect="$route.query.redirect"
            :registration-warning-message="config.registration_warning_message"
            :server-mail-configured="config.server_mail_configured"
            :session-csrf-token="sessionCsrfToken"
            :show-welcome-with-login="config.show_welcome_with_login"
            :show-reset-link="config.enable_account_interface"
            :terms-url="config.terms_url"
            :welcome-url="config.welcome_url" />
    </div>
</template>

<script>
import { getGalaxyInstance } from "app";
import ChangePassword from "components/Login/ChangePassword";
import LoginIndex from "components/Login/LoginIndex";

export default {
    components: {
        ChangePassword,
        LoginIndex,
    },
    computed: {
        config() {
            return getGalaxyInstance().config;
        },
        hasToken() {
            return this.$route.query.token || this.$route.query.expired_user;
        },
        sessionCsrfToken() {
            return getGalaxyInstance().session_csrf_token;
        },
    },
};
</script>
