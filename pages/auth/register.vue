<script lang="ts" setup>
const { locale, setLocale, t } = useI18n();
import { vMaska } from "maska/vue";
import { format, getUnixTime } from "date-fns";
import type { FormSubmitEvent } from "#ui/types";
import { number, object, string, type InferType } from "yup";
const colorMode = useColorMode();

const schema = object({
  email: string()
    .required(t("auth.errors.required"))
    .email(t("auth.errors.email")),
  password: string()
    .required(t("auth.errors.required"))
    .min(6, t("auth.errors.password"))
    .max(32, t("auth.errors.password")),
  name: string().required(t("auth.errors.required")),
  surname: string().required(t("auth.errors.required")),
  username: string().required(t("auth.errors.required")),
  birthdate: string().required(t("auth.errors.required")),
  sex: string().required(t("auth.errors.required")),
  city: string().required(t("auth.errors.required")),
  country: string().required(t("auth.errors.required")),
});

type Schema = InferType<typeof schema>;

const formState = reactive<Schema>({
  email: "",
  password: "",
  name: "",
  surname: "",
  username: "",
  // !@ts-expect-error included here bc birthdate is a string(after choosing in the form), but new Date creating number object
  // @ts-expect-error
  birthdate: new Date().setFullYear(new Date().getFullYear() - 12),
  sex: "",
  city: "",
  country: "",
});

const attrs2 = ref([
  {
    key: "today",
    highlight: {
      color: "purple",
      fillMode: "solid",
    },
    dates: new Date(),
  },
]);

const itemsS = ref([
  {
    name: t("auth.male"),
    value: "m",
  },
  {
    name: t("auth.female"),
    value: "f",
  },
  {
    name: t("auth.other"),
    value: "o",
  },
]);

const errorMessage = ref("");
const errorOn = ref(false);

const onSubmit = async (event: FormSubmitEvent<any>) => {
  errorOn.value = false;
  const { data, status, error } = await useFetch("/api/auth/register", {
    method: "POST",
    body: {
      email: formState.email,
      password: formState.password,
      name: formState.name,
      surname: formState.surname,
      username: formState.username,
      birthdate: formState.birthdate,
      sex: formState.sex,
      city: formState.city,
      country: formState.country,
    },
  });

  if (status.value == "success") {
    navigateTo("/auth/login?message=register_success");
  } else {
    // !@ts-expect-error included bc if status.value an error, it's containing an error
    //@ts-expect-error
    errorMessage.value =
      error.value?.message.split(" ")[
        error.value?.message.split(" ").length - 1
      ];
    errorOn.value = true;

    // todo: show error to user
  }
};
</script>
<template>
  <div
    class="flex items-center justify-center min-h-screen min-w-screen py-5 z-50 relative bg-white dark:bg-slate-900"
  >
    <UForm
      :state="formState"
      @submit="onSubmit"
      :schema="schema"
      class="flex flex-col gap-6"
    >
      <h1 class="exo text-4xl">{{ t("auth.register") }}</h1>
      <div class="flex flex-col gap-3">
        <UFormGroup
          :label="t('auth.username')"
          name="username"
          required
          size="lg"
        >
          <UInput
            v-model="formState.username"
            color="violet"
            variant="outline"
            placeholder="@username"
            v-maska="{ mask: '!@**************' }"
          />
        </UFormGroup>

        <UFormGroup
          :label="t('auth.password')"
          name="password"
          required
          size="lg"
        >
          <UInput
            v-model="formState.password"
            color="violet"
            variant="outline"
            type="password"
            :placeholder="t('auth.password')"
          />
        </UFormGroup>

        <UFormGroup :label="t('auth.email')" name="email" required size="lg">
          <UInput
            v-model="formState.email"
            color="violet"
            variant="outline"
            type="email"
            :placeholder="t('auth.email')"
          />
        </UFormGroup>

        <UFormGroup :label="t('auth.name')" name="name" required size="lg">
          <UInput
            v-model="formState.name"
            color="violet"
            variant="outline"
            :placeholder="t('auth.name')"
          />
        </UFormGroup>
        <UFormGroup
          :label="t('auth.surname')"
          name="surname"
          required
          size="lg"
        >
          <UInput
            v-model="formState.surname"
            color="violet"
            variant="outline"
            :placeholder="t('auth.surname')"
          />
        </UFormGroup>

        <UFormGroup
          :label="t('auth.birthdate')"
          name="birthdate"
          required
          size="lg"
        >
          <UPopover class="self-center">
            <UButton
              color="violet"
              variant="outline"
              icon="i-heroicons-calendar-days-20-solid"
              :label="format(Number(formState.birthdate), 'dd.MM.yyyy')"
            />

            <template #panel>
              <VDatePicker
                v-model="formState.birthdate"
                color="purple"
                :is-dark="colorMode.preference === 'dark'"
                :attrs="attrs2"
                :locale="locale.split('-')[0]"
                :min-date="new Date(1900, 0, 1)"
                :max-date="
                  new Date().setFullYear(new Date().getFullYear() - 12)
                "
                mode="date"
              />
            </template>
          </UPopover>
        </UFormGroup>

        <UFormGroup :label="t('auth.sex')" name="sex" required size="lg">
          <UInputMenu
            v-model="formState.sex"
            :options="itemsS"
            option-attribute="name"
            value-attribute="value"
            color="violet"
            variant="outline"
            class=""
          >
          </UInputMenu>
        </UFormGroup>

        <UFormGroup :label="t('auth.city')" name="city" required size="lg">
          <UInput
            color="violet"
            variant="outline"
            :placeholder="t('auth.city')"
            v-model="formState.city"
          />
        </UFormGroup>

        <UFormGroup
          :label="t('auth.country')"
          name="country"
          required
          size="lg"
        >
          <UInput
            color="violet"
            variant="outline"
            :placeholder="t('auth.country')"
            v-model="formState.country"
          />
        </UFormGroup>
      </div>

      <UButton
        color="violet"
        variant="solid"
        size="xl"
        type="submit"
        class="w-fit self-center"
      >
        {{ t("auth.register") }}
      </UButton>
      <div class="text-red exo" v-if="errorOn">{{ t(errorMessage) }}</div>
    </UForm>
  </div>
</template>
<style scoped></style>
