<template>
  <div
    class="min-h-screen bg-gradient-to-r from-pink-400 to-rose-400 flex items-center justify-center p-4"
  >
    <Card class="w-full max-w-md">
      <CardHeader class="text-center mb-4">
        <CardTitle class="text-2xl font-bold text-gray-800">Register</CardTitle>
      </CardHeader>
      <CardContent>
        <form @submit.prevent="handleRegister" class="space-y-4">
          <!-- First Name Field -->
          <div>
            <Label for="firstName" class="mb-2">First Name</Label>
            <Input
              id="firstName"
              v-model="form.firstName"
              type="text"
              required
              placeholder="Nama Depan Admin"
            />
          </div>
          <!-- Last Name Field -->
          <div>
            <Label for="lastName" class="mb-2">Last Name</Label>
            <Input
              id="lastName"
              v-model="form.lastName"
              type="text"
              required
              placeholder="Nama Belakang Admin"
            />
          </div>
          <!-- Email Field -->
          <div>
            <Label for="email" class="mb-2">Email</Label>
            <Input
              id="email"
              v-model="form.email"
              type="email"
              required
              placeholder="admin@tokoroti.com"
            />
          </div>
          <!-- Password Field -->
          <div>
            <Label for="password" class="mb-2">Password</Label>
            <Input
              id="password"
              v-model="form.password"
              type="password"
              required
              minlength="6"
              placeholder="••••••••"
            />
            <p class="text-xs text-gray-500 mt-1">Minimal 6 karakter</p>
          </div>
          <!-- Confirm Password Field -->
          <div>
            <Label for="confirmPassword" class="mb-2">Konfirmasi Password</Label>
            <Input
              id="confirmPassword"
              v-model="form.confirmPassword"
              type="password"
              required
              placeholder="••••••••"
            />
          </div>
          <!-- Error Message -->
          <div v-if="errorMessage" class="text-red-600 text-sm text-center">
            {{ errorMessage }}
          </div>
          <!-- Success Message -->
          <div v-if="successMessage" class="text-green-600 text-sm text-center">
            {{ successMessage }}
          </div>
          <!-- Submit Button -->
          <Button
            type="submit"
            :disabled="loading"
            class="w-full mt-6 bg-gradient-to-r from-pink-400 to-rose-400 text-white py-2 px-4 rounded-md hover:from-pink-500 hover:to-rose-600 focus:outline-none focus:ring-2 focus:ring-orange-500 focus:ring-offset-2 transition duration-200 disabled:opacity-50"
          >
            <span v-if="loading">Mendaftar...</span>
            <span v-else>Daftar Admin</span>
          </Button>
        </form>
        <!-- Login Link -->
        <div class="mt-6 text-center">
          <p class="text-gray-600 text-sm">
            Sudah punya akun admin?
            <NuxtLink
              to="/login"
              class="text-rose-500 hover:text-rose-600 font-medium"
            >
              Masuk di sini
            </NuxtLink>
          </p>
        </div>
      </CardContent>
    </Card>
  </div>
</template>

<script setup>
import { ref, reactive } from "vue";
import { useRouter } from "#app"; // Ensure definePageMeta is imported from #app
import { useAuth } from "~/composables/useAuth"; // Adjust path if necessary

// shadcn/ui components
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

// Redirect if already authenticated
definePageMeta({
  layout: false,
  middleware: "guest",
});

// Import necessary modules
const { registerUser } = useAuth();
const router = useRouter();

// Reactive form state
const form = reactive({
  firstName: "",
  lastName: "",
  email: "",
  password: "",
  confirmPassword: "",
});

// Reactive state for loading and messages
const loading = ref(false);
const errorMessage = ref("");
const successMessage = ref("");

// Handle registration logic
const handleRegister = async () => {
  loading.value = true;
  errorMessage.value = "";
  successMessage.value = "";

  if (form.password !== form.confirmPassword) {
    errorMessage.value = "Password dan konfirmasi password tidak sama";
    loading.value = false;
    return;
  }

  if (form.password.length < 6) {
    errorMessage.value = "Password minimal 6 karakter";
    loading.value = false;
    return;
  }

  try {
    await registerUser(
      form.email,
      form.password,
      form.firstName,
      form.lastName
    );

    successMessage.value = "Registrasi berhasil! Mengarahkan ke dashboard...";

    router.push("/");
  } catch (error) {
    errorMessage.value = getErrorMessage(error.code);
  } finally {
    loading.value = false;
  }
};

// Error handling function and Messages
const getErrorMessage = (errorCode) => {
  switch (errorCode) {
    case "auth/email-already-in-use":
      return "Email sudah digunakan";
    case "auth/invalid-email":
      return "Format email tidak valid";
    case "auth/weak-password":
      return "Password terlalu lemah";
    case "auth/operation-not-allowed":
      return "Registrasi tidak diizinkan";
    default:
      return "Registrasi gagal. Silakan coba lagi";
  }
};
</script>
