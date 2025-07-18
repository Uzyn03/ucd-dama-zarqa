<template>
  <div
    class="min-h-screen bg-gradient-to-r from-pink-400 to-rose-400 flex items-center justify-center p-4"
  >
    <Card class="w-full max-w-md">
      <CardHeader class="text-center mb-4">
        <CardTitle class="text-2xl font-bold text-gray-800">Login</CardTitle>
      </CardHeader>
      <CardContent>
        <form @submit.prevent="handleLogin" class="space-y-4">
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
              placeholder="••••••••"
            />
          </div>
          <!-- Error Message -->
          <div v-if="errorMessage" class="text-red-600 text-sm text-center">
            {{ errorMessage }}
          </div>
          <!-- Submit Button -->
          <Button
            type="submit"
            :disabled="loading"
            class="w-full mt-6 bg-gradient-to-r from-pink-400 to-rose-400 text-white py-2 px-4 rounded-md hover:from-pink-500 hover:to-rose-500 focus:outline-none focus:ring-2 focus:ring-orange-500 focus:ring-offset-2 transition duration-200 disabled:opacity-50"
          >
            <span v-if="loading">Memproses...</span>
            <span v-else>Masuk</span>
          </Button>
        </form>
        <!-- Register Link -->
        <div class="mt-6 text-center">
          <p class="text-gray-600 text-sm">
            Belum punya akun admin?
            <NuxtLink
              to="/register"
              class="text-rose-500 hover:text-rose-600 font-medium"
            >
              Daftar di sini
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

const { loginUser } = useAuth();
const router = useRouter();

const form = reactive({
  email: "",
  password: "",
});

const loading = ref(false);
const errorMessage = ref("");

const handleLogin = async () => {
  loading.value = true;
  errorMessage.value = "";

  try {
    await loginUser(form.email, form.password);
    router.push("/");
  } catch (error) {
    errorMessage.value = getErrorMessage(error.code);
  } finally {
    loading.value = false;
  }
};

const getErrorMessage = (errorCode) => {
  switch (errorCode) {
    case "auth/user-not-found":
      return "Email tidak ditemukan";
    case "auth/wrong-password":
      return "Password salah";
    case "auth/invalid-email":
      return "Format email tidak valid";
    case "auth/too-many-requests":
      return "Terlalu banyak percobaan. Coba lagi nanti";
    default:
      return "Login gagal. Periksa email dan password Anda";
  }
};
</script>
