<template>
  <div
    class="relative min-h-screen w-screen flex items-center justify-center bg-slate-950 overflow-hidden font-sans select-none antialiased"
  >
    <div
      class="absolute -top-12 -left-12 w-[500px] h-[500px] rounded-full bg-gradient-to-br from-blue-500 to-indigo-700 blur-[120px] opacity-40 mix-blend-screen animate-float-1"
    ></div>
    <div
      class="absolute -bottom-20 -right-12 w-[550px] h-[550px] rounded-full bg-gradient-to-tr from-pink-500 to-purple-700 blur-[140px] opacity-35 mix-blend-screen animate-float-2"
    ></div>
    <div
      class="absolute top-[25%] right-[20%] w-[350px] h-[350px] rounded-full bg-gradient-to-br from-emerald-400 to-teal-700 blur-[110px] opacity-25 mix-blend-screen animate-float-3"
    ></div>

    <div class="w-full flex items-center justify-center">
      <router-view />
    </div>
  </div>
</template>

<style>
@keyframes float {
  0% {
    transform: translate(0, 0) scale(1);
  }
  50% {
    transform: translate(30px, 20px) scale(1.05);
  }
  100% {
    transform: translate(0, 0) scale(1);
  }
}

.animate-float-1 {
  animation: float 40s infinite ease-in-out;
}
.animate-float-2 {
  animation: float 45s infinite ease-in-out;
  animation-delay: -10s;
}
.animate-float-3 {
  animation: float 35s infinite ease-in-out;
  animation-delay: -20s;
}
</style>
