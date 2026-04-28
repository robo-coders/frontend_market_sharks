<script setup lang="ts">
import { ref, reactive } from "vue";
import { Button } from "./ui/button";
import { Card, CardHeader, CardContent, CardFooter } from "./ui/card";
import { Label } from "./ui/label";
import { Input } from "./ui/input";
import {
  Select,
  SelectContent,
  SelectGroup,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "./ui/select";
import { Textarea } from "./ui/textarea";
import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert";

import { AlertCircle, Phone, Mail, Clock, MessageCircle } from "lucide-vue-next";

interface ContactFormeProps {
  firstName: string;
  lastName: string;
  email: string;
  subject: string;
  message: string;
}

const contactForm = reactive<ContactFormeProps>({
  firstName: "",
  lastName: "",
  email: "",
  subject: "General Inquiry",
  message: "",
});

const invalidInputForm = ref<boolean>(false);

const handleSubmit = () => {
  const { firstName, lastName, email, subject, message } = contactForm;
  const mailToLink = `mailto:support@marketsharks.live?subject=${subject}&body=Hello I am ${firstName} ${lastName}, my Email is ${email}. %0D%0A${message}`;
  window.location.href = mailToLink;
};
</script>

<template>
  <section id="contact" class="container py-24 sm:py-32">
    <section class="grid grid-cols-1 md:grid-cols-2 gap-8">
      <div>
        <div class="mb-4">
          <h2 class="text-lg text-primary mb-2 tracking-wider">Contact</h2>
          <h2 class="text-3xl md:text-4xl font-bold">Connect With Us</h2>
        </div>
        <p class="mb-8 text-muted-foreground lg:w-5/6">
          Have a question about our signals, plans, or community? Reach out and our team will get back to you as soon as possible.
        </p>

        <div class="flex flex-col gap-4">
          <div>
            <div class="flex gap-2 mb-1">
              <Mail />
              <div class="font-bold">Mail Us</div>
            </div>
            <div>support@marketsharks.live</div>
          </div>

          <div>
            <div class="flex gap-2 mb-1">
              <MessageCircle />
              <div class="font-bold">WhatsApp</div>
            </div>
            <div>Available for active members</div>
          </div>

          <div>
            <div class="flex gap-2 mb-1">
              <Phone />
              <div class="font-bold">Telegram</div>
            </div>
            <div>@marketsharkslive</div>
          </div>

          <div>
            <div class="flex gap-2">
              <Clock />
              <div class="font-bold">Support Hours</div>
            </div>
            <div>
              <div>Monday - Saturday</div>
              <div>9AM - 6PM GST</div>
            </div>
          </div>
        </div>
      </div>

      <!-- form -->
      <Card class="bg-muted/60 dark:bg-card">
        <CardHeader class="text-primary text-2xl"> </CardHeader>
        <CardContent>
          <form @submit.prevent="handleSubmit" class="grid gap-4">
            <div class="flex flex-col md:flex-row gap-8">
              <div class="flex flex-col w-full gap-1.5">
                <Label for="first-name">First Name</Label>
                <Input
                  id="first-name"
                  type="text"
                  placeholder="John"
                  v-model="contactForm.firstName"
                />
              </div>

              <div class="flex flex-col w-full gap-1.5">
                <Label for="last-name">Last Name</Label>
                <Input
                  id="last-name"
                  type="text"
                  placeholder="Smith"
                  v-model="contactForm.lastName"
                />
              </div>
            </div>

            <div class="flex flex-col gap-1.5">
              <Label for="email">Email</Label>
              <Input
                id="email"
                type="email"
                placeholder="you@example.com"
                v-model="contactForm.email"
              />
            </div>

            <div class="flex flex-col gap-1.5">
              <Label for="subject">Subject</Label>
              <Select v-model="contactForm.subject">
                <SelectTrigger>
                  <SelectValue placeholder="Select a subject" />
                </SelectTrigger>
                <SelectContent>
                  <SelectGroup>
                    <SelectItem value="General Inquiry">General Inquiry</SelectItem>
                    <SelectItem value="Subscription & Plans">Subscription & Plans</SelectItem>
                    <SelectItem value="Payment Issue">Payment Issue</SelectItem>
                    <SelectItem value="Signal Question">Signal Question</SelectItem>
                    <SelectItem value="Technical Support">Technical Support</SelectItem>
                  </SelectGroup>
                </SelectContent>
              </Select>
            </div>

            <div class="flex flex-col gap-1.5">
              <Label for="message">Message</Label>
              <Textarea
                id="message"
                placeholder="Your message..."
                rows="5"
                v-model="contactForm.message"
              />
            </div>

            <Alert v-if="invalidInputForm" variant="destructive">
              <AlertCircle class="w-4 h-4" />
              <AlertTitle>Error</AlertTitle>
              <AlertDescription>
                There is an error in the form. Please check your input.
              </AlertDescription>
            </Alert>

            <Button class="mt-4">Send message</Button>
          </form>
        </CardContent>
        <CardFooter></CardFooter>
      </Card>
    </section>
  </section>
</template>