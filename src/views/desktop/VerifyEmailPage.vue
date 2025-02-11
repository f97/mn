<template>
    <div class="layout-wrapper">
        <router-link to="/">
            <div class="auth-logo d-flex align-start gap-x-3">
                <img alt="logo" class="login-page-logo" :src="ezBookkeepingLogoPath" />
                <h1 class="font-weight-medium leading-normal text-2xl">{{ $t('global.app.title') }}</h1>
            </div>
        </router-link>
        <v-row no-gutters class="auth-wrapper">
            <v-col cols="12" md="8" class="d-none d-md-flex align-center justify-center position-relative">
                <div class="d-flex auth-img-footer" v-if="!isDarkMode">
                    <v-img src="img/desktop/background.svg"/>
                </div>
                <div class="d-flex auth-img-footer" v-if="isDarkMode">
                    <v-img src="img/desktop/background-dark.svg"/>
                </div>
                <div class="d-flex align-center justify-center w-100 pt-10">
                    <v-img max-width="320px" src="img/desktop/people2.svg"/>
                </div>
            </v-col>
            <v-col cols="12" md="4" class="auth-card d-flex flex-column">
                <div class="d-flex align-center justify-center h-100">
                    <v-card variant="flat" class="w-100 mt-0 px-4 pt-12" max-width="500">
                        <v-card-text>
                            <h5 class="text-h5 mb-3">{{ $t('Verify your email') }}</h5>
                            <p class="mb-0" v-if="token && loading">{{ $t('Verifying...') }}</p>
                            <p class="mb-0" v-if="token && verified">{{ $t('Email has been verified') }}</p>
                            <p class="mb-0" v-if="token && !verified && errorMessage">{{ errorMessage }}</p>
                            <p class="mb-0" v-if="!token && !email">{{ $t('Parameter Invalid') }}</p>
                            <p class="mb-0" v-if="!token && email">
                                <span>{{ $t('Account activation link has been sent to your email address:') }}</span>
                                <span class="ml-1">{{ email }}</span>
                                <span class="ml-1">{{ $t(', If you don\'t receive the mail, fill password and click the button below to resend the verify mail.') }}</span>
                            </p>
                        </v-card-text>

                        <v-card-text class="pb-0 mb-6">
                            <v-form>
                                <v-row>
                                    <v-col cols="12" v-if="!loading && !token && email && isUserVerifyEmailEnabled">
                                        <v-text-field
                                            autocomplete="password"
                                            clearable
                                            :type="isPasswordVisible ? 'text' : 'password'"
                                            :disabled="loading || resending"
                                            :label="$t('Password')"
                                            :placeholder="$t('Your password')"
                                            :append-inner-icon="isPasswordVisible ? icons.eyeSlash : icons.eye"
                                            v-model="password"
                                            @click:append-inner="isPasswordVisible = !isPasswordVisible"
                                            @keyup.enter="resendEmail"
                                        />
                                    </v-col>

                                    <v-col cols="12" v-if="!loading && !token && email && isUserVerifyEmailEnabled">
                                        <v-btn block type="submit" :disabled="loading || resending || !password" @click="resendEmail">
                                            {{ $t('Resend Validation Email') }}
                                            <v-progress-circular indeterminate size="24" class="ml-2" v-if="resending"></v-progress-circular>
                                        </v-btn>
                                    </v-col>

                                    <v-col cols="12">
                                        <router-link class="d-flex align-center justify-center" to="/login"
                                                     :class="{ 'disabled': loading || resending }">
                                            <v-icon :icon="icons.left"/>
                                            <span>{{ $t('Back to log in') }}</span>
                                        </router-link>
                                    </v-col>
                                </v-row>
                            </v-form>
                        </v-card-text>
                    </v-card>
                </div>
                <v-spacer/>
                <div class="d-flex align-center justify-center">
                    <v-card variant="flat" class="w-100 px-4 pb-4" max-width="500">
                        <v-card-text class="pt-0">
                            <v-row>
                                <v-col cols="12" class="text-center">
                                    <v-menu location="bottom">
                                        <template #activator="{ props }">
                                            <v-btn variant="text"
                                                   :disabled="resending"
                                                   v-bind="props">{{ currentLanguageName }}</v-btn>
                                        </template>
                                        <v-list>
                                            <v-list-item v-for="(lang, locale) in allLanguages" :key="locale">
                                                <v-list-item-title
                                                    class="cursor-pointer"
                                                    @click="changeLanguage(locale)">
                                                    {{ lang.displayName }}
                                                </v-list-item-title>
                                            </v-list-item>
                                        </v-list>
                                    </v-menu>
                                </v-col>

                                <v-col cols="12" class="d-flex align-center pt-0">
                                    <v-divider />
                                </v-col>

                                <v-col cols="12" class="text-center text-sm">
                                    <span>Powered by </span>
                                    <a href="https://github.com/f97/n" target="_blank">ezBookkeeping</a>&nbsp;<span>{{ version }}</span>
                                </v-col>
                            </v-row>
                        </v-card-text>
                    </v-card>
                </div>
            </v-col>
        </v-row>

        <confirm-dialog ref="confirmDialog"/>
        <snack-bar ref="snackbar" @update:show="onSnackbarShowStateChanged" />
    </div>
</template>

<script>
import { useTheme } from 'vuetify';

import { mapStores } from 'pinia';
import { useRootStore } from '@/stores/index.js';
import { useSettingsStore } from '@/stores/setting.js';

import assetConstants from '@/consts/asset.js';
import { isUserVerifyEmailEnabled } from '@/lib/server_settings.js';

import {
    mdiChevronLeft,
    mdiEyeOffOutline,
    mdiEyeOutline
} from '@mdi/js';

export default {
    props: [
        'email',
        'token'
    ],
    data() {
        return {
            password: '',
            isPasswordVisible: false,
            loading: true,
            resending: false,
            verified: false,
            errorMessage: '',
            icons: {
                left: mdiChevronLeft,
                eye: mdiEyeOutline,
                eyeSlash: mdiEyeOffOutline
            }
        };
    },
    computed: {
        ...mapStores(useRootStore, useSettingsStore),
        ezBookkeepingLogoPath() {
            return assetConstants.ezBookkeepingLogoPath;
        },
        version() {
            return 'v' + this.$version;
        },
        allLanguages() {
            return this.$locale.getAllLanguageInfos();
        },
        isDarkMode() {
            return this.globalTheme.global.name.value === 'dark';
        },
        currentLanguageName() {
            return this.$locale.getCurrentLanguageDisplayName();
        },
        isUserVerifyEmailEnabled() {
            return isUserVerifyEmailEnabled();
        }
    },
    setup() {
        const theme = useTheme();

        return {
            globalTheme: theme
        };
    },
    created() {
        const self = this;

        self.verified = false;
        self.loading = true;

        if (!self.token) {
            self.loading = false;
            return;
        }

        self.rootStore.verifyEmail({
            token: self.token,
            requestNewToken: !self.$user.isUserLogined()
        }).then(() => {
            self.loading = false;
            self.verified = true;
            self.$refs.snackbar.showMessage('Email has been verified');
        }).catch(error => {
            self.loading = false;
            self.verified = false;

            if (!error.processed) {
                self.errorMessage = self.$tError(error.message || error);
                self.$refs.snackbar.showError(error);
            }
        });
    },
    methods: {
        resendEmail() {
            const self = this;

            self.resending = true;

            self.rootStore.resendVerifyEmailByUnloginUser({
                email: self.email,
                password: self.password
            }).then(() => {
                self.resending = false;
                self.$refs.snackbar.showMessage('Validation email has been sent');
            }).catch(error => {
                self.resending = false;

                if (!error.processed) {
                    self.$refs.snackbar.showError(error);
                }
            });
        },
        onSnackbarShowStateChanged(newValue) {
            if (!newValue && this.verified && this.$user.isUserLogined()) {
                this.$router.replace('/');
            }
        },
        changeLanguage(locale) {
            const localeDefaultSettings = this.$locale.setLanguage(locale);
            this.settingsStore.updateLocalizedDefaultSettings(localeDefaultSettings);
        }
    }
}
</script>
