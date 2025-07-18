<template>
  <div class="container mx-auto px-4 min-w-full">
    <!-- Header -->
    <div class="flex items-center justify-between">
      <HeadersContent
        title="Expense Management"
        description="Track and manage your business expenses"
      />
      <div class="flex space-x-2">
        <Button variant="outline" @click="showFilters = !showFilters">
          <Filter class="mr-2 h-4 w-4" />
          Filters
        </Button>
        <Button variant="outline" @click="exportExpenses">
          <Download class="mr-2 h-4 w-4" />
          Export
        </Button>
        <Button @click="showAddExpenseModal = true">
          <Plus class="mr-2 h-4 w-4" />
          Add Expense
        </Button>
      </div>
    </div>

    <!-- Alert Messages -->
    <Alert
      v-if="message"
      :class="messageType === 'error' ? 'border-red-500' : 'border-green-500'"
      class="mb-6"
    >
      <AlertDescription>{{ message }}</AlertDescription>
    </Alert>

    <!-- Filters -->
    <Card v-if="showFilters" class="mb-6">
      <CardContent class="pt-6">
        <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
          <div>
            <Label class="mb-2">Category</Label>
            <Select v-model="filters.category">
              <SelectTrigger>
                <SelectValue placeholder="All Categories" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="all">All Categories</SelectItem>
                <SelectItem value="materials">Raw Materials</SelectItem>
                <SelectItem value="labor">Labor Costs</SelectItem>
                <SelectItem value="operational">Operational</SelectItem>
                <SelectItem value="marketing">Marketing</SelectItem>
                <SelectItem value="utilities">Utilities</SelectItem>
                <SelectItem value="transportation">Transportation</SelectItem>
                <SelectItem value="others">Others</SelectItem>
              </SelectContent>
            </Select>
          </div>
          <div>
            <Label class="mb-2">Status</Label>
            <Select v-model="filters.status">
              <SelectTrigger>
                <SelectValue placeholder="All Status" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="all">All Status</SelectItem>
                <SelectItem value="pending">Pending</SelectItem>
                <SelectItem value="approved">Approved</SelectItem>
                <SelectItem value="paid">Paid</SelectItem>
                <SelectItem value="rejected">Rejected</SelectItem>
              </SelectContent>
            </Select>
          </div>
          <div>
            <Label class="mb-2">Date From</Label>
            <Input type="date" v-model="filters.dateFrom" />
          </div>
          <div>
            <Label class="mb-2">Date To</Label>
            <Input type="date" v-model="filters.dateTo" />
          </div>
        </div>
        <div class="flex justify-between items-center mt-4">
          <div class="flex items-center space-x-2">
            <Label>Search:</Label>
            <Input
              v-model="filters.search"
              placeholder="Search expenses..."
              class="w-64"
            />
          </div>
          <div class="flex space-x-2">
            <Button variant="outline" @click="clearFilters">Clear</Button>
            <Button @click="applyFilters">Apply Filters</Button>
          </div>
        </div>
      </CardContent>
    </Card>

    <!-- Expense Summary Cards -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
      <Card>
        <CardHeader
          class="flex flex-row items-center justify-between space-y-0"
        >
          <CardTitle class="text-sm font-medium">Total Expenses</CardTitle>
          <DollarSign class="text-red-500" />
        </CardHeader>
        <CardContent>
          <div class="text-xl font-bold text-red-600">
            Rp {{ formatPrice(expenseSummary.total) }}
          </div>
          <p class="text-xs text-muted-foreground">This month</p>
        </CardContent>
      </Card>
      <Card>
        <CardHeader
          class="flex flex-row items-center justify-between space-y-0"
        >
          <CardTitle class="text-sm font-medium">Pending Approval</CardTitle>
          <Clock />
        </CardHeader>
        <CardContent>
          <div class="text-xl font-bold text-orange-600">
            {{ expenseSummary.pending }}
          </div>
          <p class="text-xs text-muted-foreground">Awaiting approval</p>
        </CardContent>
      </Card>
      <Card>
        <CardHeader
          class="flex flex-row items-center justify-between space-y-0"
        >
          <CardTitle class="text-sm font-medium">Approved</CardTitle>
          <CheckCircle />
        </CardHeader>
        <CardContent>
          <div class="text-xl font-bold text-green-600">
            {{ expenseSummary.approved }}
          </div>
          <p class="text-xs text-muted-foreground">Ready for payment</p>
        </CardContent>
      </Card>
      <Card>
        <CardHeader
          class="flex flex-row items-center justify-between space-y-0"
        >
          <CardTitle class="text-sm font-medium">Paid</CardTitle>
          <CreditCard />
        </CardHeader>
        <CardContent>
          <div class="text-xl font-bold text-blue-600">
            {{ expenseSummary.paid }}
          </div>
          <p class="text-xs text-muted-foreground">Completed</p>
        </CardContent>
      </Card>
    </div>

    <!-- Expenses Table -->
    <Card class="mb-4">
      <CardHeader class="flex flex-row items-center justify-between">
        <div>
          <CardTitle>Expense Records</CardTitle>
          <CardDescription>
            Showing {{ filteredExpenses.length }} of
            {{ expenses.length }} expenses
          </CardDescription>
        </div>
        <div class="flex items-center space-x-2">
          <Select v-model="itemsPerPage">
            <SelectTrigger class="w-20">
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              <SelectItem value="10">10</SelectItem>
              <SelectItem value="25">25</SelectItem>
              <SelectItem value="50">50</SelectItem>
              <SelectItem value="100">100</SelectItem>
            </SelectContent>
          </Select>
          <span class="text-sm text-muted-foreground">per page</span>
        </div>
      </CardHeader>
      <CardContent>
        <div v-if="loadingExpenses" class="text-center py-8">
          <div
            class="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600 mx-auto"
          ></div>
          <p class="mt-2 text-sm text-muted-foreground">Loading expenses...</p>
        </div>
        <Table v-else>
          <TableHeader>
            <TableRow>
              <TableHead>Expense ID</TableHead>
              <TableHead>Description</TableHead>
              <TableHead>Category</TableHead>
              <TableHead>Amount</TableHead>
              <TableHead>Date</TableHead>
              <TableHead>Submitted By</TableHead>
              <TableHead>Status</TableHead>
              <TableHead>Actions</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            <TableRow v-for="expense in paginatedExpenses" :key="expense.id">
              <TableCell class="font-mono">{{ expense.expenseId }}</TableCell>
              <TableCell>
                <div class="flex items-center space-x-3">
                  <div class="p-2 rounded-full bg-red-100">
                    <component
                      :is="getCategoryIcon(expense.category)"
                      class="h-4 w-4 text-red-600"
                    />
                  </div>
                  <div>
                    <div class="font-medium">{{ expense.description }}</div>
                    <div
                      v-if="expense.vendor"
                      class="text-sm text-muted-foreground"
                    >
                      Vendor: {{ expense.vendor }}
                    </div>
                  </div>
                </div>
              </TableCell>
              <TableCell>
                <Badge variant="outline">{{
                  getCategoryLabel(expense.category)
                }}</Badge>
              </TableCell>
              <TableCell class="font-bold text-red-600"
                >-Rp {{ formatPrice(expense.amount) }}</TableCell
              >
              <TableCell>
                <div class="text-sm">
                  {{ formatDate(expense.date) }}
                </div>
                <div class="text-xs text-muted-foreground">
                  {{ formatTime(expense.date) }}
                </div>
              </TableCell>
              <TableCell>{{ expense.submittedBy }}</TableCell>
              <TableCell>
                <Badge :variant="getStatusVariant(expense.status)">
                  {{ expense.status }}
                </Badge>
              </TableCell>
              <TableCell>
                <div class="flex space-x-2">
                  <Button
                    variant="ghost"
                    size="sm"
                    @click="viewExpense(expense)"
                  >
                    <Eye class="h-4 w-4" />
                  </Button>
                  <Button
                    variant="ghost"
                    size="sm"
                    @click="editExpense(expense)"
                  >
                    <Edit class="h-4 w-4" />
                  </Button>
                  <Button
                    variant="ghost"
                    size="sm"
                    @click="deleteExpense(expense.id)"
                    class="text-red-600"
                  >
                    <Trash2 class="h-4 w-4" />
                  </Button>
                </div>
              </TableCell>
            </TableRow>
          </TableBody>
        </Table>

        <!-- Pagination -->
        <div class="flex items-center justify-between mt-4">
          <div class="text-sm text-muted-foreground">
            Showing {{ (currentPage - 1) * parseInt(itemsPerPage) + 1 }} to
            {{
              Math.min(
                currentPage * parseInt(itemsPerPage),
                filteredExpenses.length
              )
            }}
            of {{ filteredExpenses.length }} results
          </div>
          <div class="flex space-x-2">
            <Button
              variant="outline"
              size="sm"
              @click="previousPage"
              :disabled="currentPage === 1"
            >
              <ChevronLeft class="h-4 w-4" />
              Previous
            </Button>
            <Button
              variant="outline"
              size="sm"
              @click="nextPage"
              :disabled="currentPage === totalPages"
            >
              Next
              <ChevronRight class="h-4 w-4" />
            </Button>
          </div>
        </div>
      </CardContent>
    </Card>

    <!-- Add/Edit Expense Modal -->
    <Dialog v-model:open="showAddExpenseModal" :key="modalKey">
      <DialogContent class="max-w-2xl">
        <DialogHeader>
          <DialogTitle>{{
            editingExpense ? "Edit Expense" : "Add New Expense"
          }}</DialogTitle>
        </DialogHeader>
        <form @submit.prevent="saveExpense" class="space-y-4">
          <!-- Loading Status -->
          <div v-if="uploadingStatus" class="text-center py-4">
            <div
              class="animate-spin rounded-full h-6 w-6 border-b-2 border-black mx-auto"
            ></div>
            <p class="mt-2 text-sm text-muted-foreground">
              {{ uploadingStatus }}
            </p>
          </div>

          <div class="grid grid-cols-2 gap-4">
            <div>
              <Label for="description" class="mb-2">Description *</Label>
              <Input
                id="description"
                v-model="expenseForm.description"
                :disabled="isLoading"
                required
              />
            </div>
            <div>
              <Label for="category" class="mb-2">Category *</Label>
              <Select
                v-model="expenseForm.category"
                :disabled="isLoading"
                required
              >
                <SelectTrigger>
                  <SelectValue placeholder="Select category" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="materials">Raw Materials</SelectItem>
                  <SelectItem value="labor">Labor Costs</SelectItem>
                  <SelectItem value="operational">Operational</SelectItem>
                  <SelectItem value="marketing">Marketing</SelectItem>
                  <SelectItem value="utilities">Utilities</SelectItem>
                  <SelectItem value="transportation">Transportation</SelectItem>
                  <SelectItem value="others">Others</SelectItem>
                </SelectContent>
              </Select>
            </div>
          </div>
          <div class="grid grid-cols-2 gap-4">
            <div>
              <Label for="amount" class="mb-2">Amount (Rp) *</Label>
              <Input
                id="amount"
                type="number"
                v-model="expenseForm.amount"
                :disabled="isLoading"
                required
              />
            </div>
            <div>
              <Label for="date" class="mb-2">Date *</Label>
              <Input
                id="date"
                type="date"
                v-model="expenseForm.date"
                :disabled="isLoading"
                required
              />
            </div>
          </div>
          <div class="grid grid-cols-2 gap-4">
            <div>
              <Label for="vendor" class="mb-2">Vendor/Supplier</Label>
              <Input
                id="vendor"
                v-model="expenseForm.vendor"
                :disabled="isLoading"
              />
            </div>
            <div>
              <Label for="paymentMethod" class="mb-2">Payment Method</Label>
              <Select v-model="expenseForm.paymentMethod" :disabled="isLoading">
                <SelectTrigger>
                  <SelectValue placeholder="Select payment method" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="cash">Cash</SelectItem>
                  <SelectItem value="bank_transfer">Bank Transfer</SelectItem>
                  <SelectItem value="credit_card">Credit Card</SelectItem>
                  <SelectItem value="check">Check</SelectItem>
                </SelectContent>
              </Select>
            </div>
          </div>
          <div>
            <Label for="notes" class="mb-2">Notes</Label>
            <Textarea
              id="notes"
              v-model="expenseForm.notes"
              :disabled="isLoading"
              rows="3"
            />
          </div>
          <div>
            <Label for="receipt" class="mb-2">Receipt/Invoice</Label>
            <Input
              id="receipt"
              type="file"
              accept="image/*,application/pdf"
              :disabled="isLoading"
              @change="handleFileUpload"
            />
            <p class="text-sm text-muted-foreground mt-1">
              Maximum file size: 5MB. Supported formats: JPG, PNG, PDF
            </p>
          </div>

          <div class="flex justify-end space-x-2">
            <Button
              type="button"
              variant="outline"
              :disabled="isLoading"
              @click="handleCancelExpense"
            >
              Cancel
            </Button>
            <Button type="submit" :disabled="isLoading">
              {{ editingExpense ? "Update" : "Save" }} Expense
            </Button>
          </div>
        </form>
      </DialogContent>
    </Dialog>

    <!-- View Modal -->
    <Dialog v-model:open="showViewExpenseModal">
      <DialogContent class="max-w-xl">
        <DialogHeader>
          <DialogTitle>Expense Details</DialogTitle>
        </DialogHeader>
        <div v-if="selectedExpense" class="space-y-4">
          <div class="grid grid-cols-2 gap-4">
            <div>
              <Label class="font-medium mb-2">Expense ID</Label>
              <p class="font-mono">{{ selectedExpense.expenseId }}</p>
            </div>
            <div>
              <Label class="font-medium mb-2">Status</Label>
              <Badge :variant="getStatusVariant(selectedExpense.status)">
                {{ selectedExpense.status }}
              </Badge>
            </div>
          </div>
          <div class="grid grid-cols-2 gap-4">
            <div>
              <Label class="font-medium mb-2">Amount</Label>
              <p class="text-lg font-bold text-red-600">
                -Rp {{ formatPrice(selectedExpense.amount) }}
              </p>
            </div>
            <div>
              <Label class="font-medium mb-2">Category</Label>
              <p>{{ getCategoryLabel(selectedExpense.category) }}</p>
            </div>
          </div>
          <div class="grid grid-cols-2 gap-4">
            <div>
              <Label class="font-medium mb-2">Description</Label>
              <p>{{ selectedExpense.description }}</p>
            </div>
            <div>
              <Label class="font-medium mb-2">Vendor/Supplier</Label>
              <p>{{ selectedExpense.vendor || "-" }}</p>
            </div>
          </div>
          <div class="grid grid-cols-2 gap-4">
            <div>
              <Label class="font-medium mb-2">Date</Label>
              <p>
                {{ formatDate(selectedExpense.date) }}
                {{ formatTime(selectedExpense.date) }}
              </p>
            </div>
            <div>
              <Label class="font-medium mb-2">Payment Method</Label>
              <p>{{ selectedExpense.paymentMethod || "-" }}</p>
            </div>
          </div>
          <div v-if="selectedExpense.notes">
            <Label class="font-medium mb-2">Notes</Label>
            <p class="text-sm text-muted-foreground">
              {{ selectedExpense.notes }}
            </p>
          </div>
          <div v-if="selectedExpense.receiptUrl">
            <Label class="font-medium mb-2">Receipt/Invoice</Label>
            <a
              :href="selectedExpense.receiptUrl"
              target="_blank"
              class="text-blue-600 underline"
              >View Receipt</a
            >
          </div>
        </div>
      </DialogContent>
    </Dialog>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, watch, nextTick } from "vue";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Alert, AlertDescription } from "@/components/ui/alert";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import { Badge } from "@/components/ui/badge";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";
import {
  Plus,
  Filter,
  Download,
  Eye,
  Edit,
  Trash2,
  DollarSign,
  Clock,
  CheckCircle,
  CreditCard,
  ChevronLeft,
  ChevronRight,
  Truck,
  Users,
  Settings,
  Smartphone,
  Zap,
  Car,
  FileText,
} from "lucide-vue-next";
import {
  collection,
  getDocs,
  addDoc,
  serverTimestamp,
  query,
  where,
  orderBy,
  limit,
  doc,
  setDoc,
  deleteDoc,
} from "firebase/firestore";
import HeadersContent from "~/components/ui/HeadersContent.vue";

definePageMeta({
  middleware: "auth",
});

const config = useRuntimeConfig();
const { user } = useAuth();

// State
const showFilters = ref(false);
const showAddExpenseModal = ref(false);
const editingExpense = ref(null);
const uploadingStatus = ref("");
const message = ref("");
const messageType = ref("");
const isLoading = ref(false);
const loadingExpenses = ref(false);
const selectedExpense = ref(null);
const showViewExpenseModal = ref(false);
const currentPage = ref(1);
const itemsPerPage = ref("25");
const modalKey = ref(0);

const filters = reactive({
  category: "all",
  status: "all",
  dateFrom: "",
  dateTo: "",
  search: "",
});

// State for expense form
const expenseForm = reactive({
  expenseId: "",
  description: "",
  category: "",
  amount: "",
  date: "",
  vendor: "",
  paymentMethod: "",
  notes: "",
  receiptFile: null,
});

const expenseSummary = reactive({
  total: 0,
  pending: 0,
  approved: 0,
  paid: 0,
});

const expenses = ref([]);

// Function to upload receipt to Cloudinary
const uploadToCloudinary = async (file) => {
  try {
    const formData = new FormData();
    formData.append("file", file);
    formData.append("upload_preset", config.public.cloudinaryUploadPreset);
    formData.append("folder", "TokoKueDlillah/expenses/receipts");

    const timestamp = new Date().getTime();
    const filename = `receipt_${timestamp}_${file.name.replace(
      /[^a-zA-Z0-9.]/g,
      "_"
    )}`;
    formData.append(
      "public_id",
      `TokoKueDlillah/expenses/receipts/${filename}`
    );

    const response = await fetch(
      `https://api.cloudinary.com/v1_1/${config.public.cloudinaryCloudName}/image/upload`,
      {
        method: "POST",
        body: formData,
      }
    );

    if (!response.ok) {
      const errorText = await response.text();
      throw new Error(`HTTP error! status: ${response.status} - ${errorText}`);
    }

    const data = await response.json();

    if (data.error) {
      throw new Error(data.error.message);
    }

    return data.secure_url;
  } catch (error) {
    console.error("Cloudinary upload error:", error);
    throw new Error(`Failed to upload receipt: ${error.message}`);
  }
};

// Function to generate expense ID with yearly format
const generateExpenseId = async () => {
  try {
    const { $firebase } = useNuxtApp();
    const currentYear = new Date().getFullYear();
    const yearPrefix = `EXP-${currentYear}`;

    // Query expenses from current year
    const expensesQuery = query(
      collection($firebase.firestore, "expenses"),
      where("expenseId", ">=", `${yearPrefix}-001`),
      where("expenseId", "<", `${yearPrefix}-999`),
      orderBy("expenseId", "desc"),
      limit(1)
    );

    const querySnapshot = await getDocs(expensesQuery);

    let nextNumber = 1;

    if (!querySnapshot.empty) {
      const lastExpense = querySnapshot.docs[0].data();
      const lastId = lastExpense.expenseId;
      const parts = lastId.split("-");
      if (parts.length === 3) {
        const lastNumber = parseInt(parts[2]) || 0;
        nextNumber = lastNumber + 1;
      }
    }

    // Format: EXP-2025-001
    const formattedNumber = nextNumber.toString().padStart(3, "0");
    return `${yearPrefix}-${formattedNumber}`;
  } catch (error) {
    console.error("Error generating expense ID:", error);
    // Fallback with timestamp
    return `EXP-${new Date().getFullYear()}-${Date.now().toString().slice(-3)}`;
  }
};

// Function to fetch expenses from Firestore
const fetchExpenses = async () => {
  try {
    loadingExpenses.value = true;
    const { $firebase } = useNuxtApp();

    const expensesQuery = query(
      collection($firebase.firestore, "expenses"),
      orderBy("createdAt", "desc")
    );

    const querySnapshot = await getDocs(expensesQuery);

    expenses.value = querySnapshot.docs.map((doc) => {
      const data = doc.data();
      return {
        id: doc.id,
        ...data,
        // Convert Firestore timestamp to Date
        date: data.date?.toDate ? data.date.toDate() : new Date(data.date),
        createdAt: data.createdAt?.toDate
          ? data.createdAt.toDate()
          : new Date(),
      };
    });

    // Calculate summary
    calculateExpenseSummary();

    console.log("Fetched expenses:", expenses.value);
  } catch (error) {
    console.error("Error fetching expenses:", error);
    showMessage("Failed to load expenses", "error");
  } finally {
    loadingExpenses.value = false;
  }
};

// Function to calculate expense summary
const calculateExpenseSummary = () => {
  const currentMonth = new Date().getMonth();
  const currentYear = new Date().getFullYear();

  const thisMonthExpenses = expenses.value.filter((expense) => {
    const expenseDate = new Date(expense.date);
    return (
      expenseDate.getMonth() === currentMonth &&
      expenseDate.getFullYear() === currentYear
    );
  });

  expenseSummary.total = thisMonthExpenses.reduce(
    (total, expense) => total + expense.amount,
    0
  );
  expenseSummary.pending = expenses.value.filter(
    (expense) => expense.status === "pending"
  ).length;
  expenseSummary.approved = expenses.value.filter(
    (expense) => expense.status === "approved"
  ).length;
  expenseSummary.paid = expenses.value.filter(
    (expense) => expense.status === "paid"
  ).length;
};

// Computed
const filteredExpenses = computed(() => {
  let filtered = expenses.value;

  if (filters.category !== "all") {
    filtered = filtered.filter(
      (expense) => expense.category === filters.category
    );
  }
  if (filters.status !== "all") {
    filtered = filtered.filter((expense) => expense.status === filters.status);
  }
  if (filters.dateFrom) {
    filtered = filtered.filter(
      (expense) => new Date(expense.date) >= new Date(filters.dateFrom)
    );
  }
  if (filters.dateTo) {
    filtered = filtered.filter(
      (expense) => new Date(expense.date) <= new Date(filters.dateTo)
    );
  }
  if (filters.search) {
    const search = filters.search.toLowerCase();
    filtered = filtered.filter(
      (expense) =>
        expense.description.toLowerCase().includes(search) ||
        expense.expenseId.toLowerCase().includes(search) ||
        (expense.vendor && expense.vendor.toLowerCase().includes(search))
    );
  }

  return filtered;
});

const totalPages = computed(() => {
  return Math.ceil(
    filteredExpenses.value.length / parseInt(itemsPerPage.value)
  );
});

const paginatedExpenses = computed(() => {
  const start = (currentPage.value - 1) * parseInt(itemsPerPage.value);
  const end = start + parseInt(itemsPerPage.value);
  return filteredExpenses.value.slice(start, end);
});

// Methods
const formatPrice = (price) => {
  if (!price) return "0";
  return new Intl.NumberFormat("id-ID").format(price);
};

const formatDate = (date) => {
  return new Date(date).toLocaleDateString("id-ID", {
    year: "numeric",
    month: "short",
    day: "numeric",
  });
};

const formatTime = (date) => {
  return new Date(date).toLocaleTimeString("id-ID", {
    hour: "2-digit",
    minute: "2-digit",
  });
};

const getCategoryLabel = (category) => {
  const labels = {
    materials: "Raw Materials",
    labor: "Labor Costs",
    operational: "Operational",
    marketing: "Marketing",
    utilities: "Utilities",
    transportation: "Transportation",
    others: "Others",
  };
  return labels[category] || category;
};

const getCategoryIcon = (category) => {
  const icons = {
    materials: Truck,
    labor: Users,
    operational: Settings,
    marketing: Smartphone,
    utilities: Zap,
    transportation: Car,
    others: FileText,
  };
  return icons[category] || FileText;
};

const getStatusVariant = (status) => {
  const variants = {
    pending: "secondary",
    approved: "default",
    paid: "outline",
    rejected: "destructive",
  };
  return variants[status] || "secondary";
};

const showMessage = (msg, type) => {
  message.value = msg;
  messageType.value = type;
  setTimeout(() => {
    message.value = "";
    messageType.value = "";
  }, 5000);
};

const clearFilters = () => {
  Object.assign(filters, {
    category: "all",
    status: "all",
    dateFrom: "",
    dateTo: "",
    search: "",
  });
  currentPage.value = 1;
};

const applyFilters = () => {
  currentPage.value = 1;
  showFilters.value = false;
};

const previousPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
};

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
  }
};

const viewExpense = (expense) => {
  selectedExpense.value = expense;
  showViewExpenseModal.value = true;
};

const editExpense = (expense) => {
  editingExpense.value = expense;
  Object.assign(expenseForm, {
    description: expense.description,
    category: expense.category,
    amount: expense.amount,
    date: expense.date.toISOString().split("T")[0],
    vendor: expense.vendor || "",
    paymentMethod: expense.paymentMethod || "",
    notes: expense.notes || "",
  });
  showAddExpenseModal.value = true;
};

const deleteExpense = async (expenseId) => {
  if (confirm("Are you sure you want to delete this expense?")) {
    try {
      const { $firebase } = useNuxtApp();
      await deleteDoc(doc($firebase.firestore, "expenses", expenseId));

      // Hapus dari lokal
      expenses.value = expenses.value.filter((e) => e.id !== expenseId);
      calculateExpenseSummary();

      showMessage("Expense deleted successfully!", "success");
    } catch (error) {
      console.error("Delete error:", error);
      showMessage("Failed to delete expense", "error");
    }
  }
};

const handleFileUpload = (event) => {
  const file = event.target.files[0];
  if (!file) return;

  // Validate file size (5MB)
  if (file.size > 5 * 1024 * 1024) {
    showMessage("File size must be less than 5MB", "error");
    return;
  }

  // Validate file type
  const allowedTypes = [
    "image/jpeg",
    "image/png",
    "image/jpg",
    "application/pdf",
  ];

  if (!allowedTypes.includes(file.type)) {
    showMessage("Please select a valid image (JPG, PNG) or PDF file", "error");
    return;
  }

  expenseForm.receiptFile = file;
};

const validateForm = () => {
  if (
    !expenseForm.description ||
    !expenseForm.category ||
    !expenseForm.amount ||
    !expenseForm.date
  ) {
    showMessage("Please fill in all required fields", "error");
    return false;
  }

  if (expenseForm.amount <= 0) {
    showMessage("Amount must be greater than 0", "error");
    return false;
  }

  return true;
};

const saveExpense = async () => {
  console.log("Starting saveExpense...");

  if (!validateForm()) {
    console.log("Form validation failed");
    return;
  }

  isLoading.value = true;

  try {
    const { $firebase } = useNuxtApp();

    // Generate expense ID
    const expenseId = await generateExpenseId();
    console.log("Generated expense ID:", expenseId);

    // Upload receipt if exists
    let receiptUrl = "";
    if (expenseForm.receiptFile) {
      try {
        uploadingStatus.value = "Uploading receipt...";
        receiptUrl = await uploadToCloudinary(expenseForm.receiptFile);
        console.log("Receipt uploaded successfully:", receiptUrl);
      } catch (error) {
        console.error("Receipt upload failed:", error);
        showMessage(`Failed to upload receipt: ${error.message}`, "error");
        return;
      }
    }

    // Prepare expense data
    uploadingStatus.value = "Saving expense...";

    if (editingExpense.value) {
      // Update existing expense
      const expenseData = {
        description: expenseForm.description,
        category: expenseForm.category,
        amount: Number(expenseForm.amount),
        date: new Date(expenseForm.date),
        vendor: expenseForm.vendor || "",
        paymentMethod: expenseForm.paymentMethod || "",
        notes: expenseForm.notes || "",
        receiptUrl: receiptUrl || editingExpense.value.receiptUrl,
        updatedAt: serverTimestamp(),
      };

      await setDoc(
        doc($firebase.firestore, "expenses", editingExpense.value.id),
        expenseData,
        { merge: true }
      );

      // Update local state
      const index = expenses.value.findIndex(
        (e) => e.id === editingExpense.value.id
      );
      if (index !== -1) {
        expenses.value[index] = {
          ...expenses.value[index],
          ...expenseData,
          date: new Date(expenseForm.date),
          updatedAt: new Date(),
        };
      }

      showMessage("Expense updated successfully!", "success");
    } else {
      // Add new expense
      const expenseData = {
        expenseId: expenseId,
        description: expenseForm.description,
        category: expenseForm.category,
        amount: Number(expenseForm.amount),
        date: new Date(expenseForm.date),
        vendor: expenseForm.vendor || "",
        paymentMethod: expenseForm.paymentMethod || "",
        notes: expenseForm.notes || "",
        receiptUrl: receiptUrl,
        submittedBy:
          user.value?.firstName + " " + user.value?.lastName ||
          user.value?.email ||
          "Unknown",
        status: "pending",
        createdAt: serverTimestamp(),
        updatedAt: serverTimestamp(),
      };

      console.log("Saving new expense data:", expenseData);

      const docRef = await addDoc(
        collection($firebase.firestore, "expenses"),
        expenseData
      );

      // Add to local state
      const newExpense = {
        id: docRef.id,
        ...expenseData,
        date: new Date(expenseForm.date),
        createdAt: new Date(),
      };

      expenses.value.unshift(newExpense);
      showMessage("Expense added successfully!", "success");
    }

    calculateExpenseSummary();

    // Close modal dengan delay kecil untuk memastikan message muncul
    setTimeout(async () => {
      await closeExpenseModal();
    }, 500);
  } catch (error) {
    console.error("Error saving expense:", error);
    showMessage("Failed to save expense. Please try again.", "error");
  } finally {
    isLoading.value = false;
    uploadingStatus.value = "";
  }
};

const closeExpenseModal = async () => {
  console.log("Closing expense modal...");

  // Force close modal
  showAddExpenseModal.value = false;
  editingExpense.value = null;
  uploadingStatus.value = "";

  // Reset form
  Object.assign(expenseForm, {
    expenseId: "",
    description: "",
    category: "",
    amount: "",
    date: "",
    vendor: "",
    paymentMethod: "",
    notes: "",
    receiptFile: null,
  });

  // Reset file input dengan nextTick
  await nextTick();
  const fileInput = document.getElementById("receipt");
  if (fileInput) {
    fileInput.value = "";
  }

  modalKey.value++;
  console.log("Modal closed, showAddExpenseModal:", showAddExpenseModal.value);
};

const handleCancelExpense = async () => {
  console.log("Cancel button clicked");
  await closeExpenseModal();
};

const exportExpenses = () => {
  console.log("Exporting expenses...");
};

watch(showAddExpenseModal, (newVal, oldVal) => {
  console.log(`Modal state changed from ${oldVal} to ${newVal}`);
});

onMounted(() => {
  console.log("Expense management loaded");
  fetchExpenses();
});
</script>
