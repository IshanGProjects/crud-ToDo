<script>
  import { onMount } from "svelte";
  import { auth, db } from "../lib/firebase/firebase";
  import { doc, getDoc, setDoc } from "firebase/firestore";
  import { authStore } from "../store/store";

  const nonAuthRoutes = ["/"];

  onMount(() => {
    console.log("Mounting...");
    const unsubscribe = auth.onAuthStateChanged(async (user) => {
      const currentPath = window.location.pathname;

      if (!user && !nonAuthRoutes.includes(currentPath)) {
        window.location.href = "/";
        return;
      }

      //We do have auth at this point
      if (user && currentPath === "/") {
        window.location.href = "/dashboard";
        return;
      }

      if (!user) {
        return;
      }

      let dataToStore;
      //Now we have user existence fetch data
      const docRef = doc(db, "users", user.uid);
      const docSnap = await getDoc(docRef);
      if (docSnap.exists()) {
        const userRef = doc(db, "user", user.uid);
        dataToStore = {
          email: user.email,
          todos: [],
        };
        await setDoc(userRef, dataToStore, { merge: true });
      } else {
        const userData = docSnap.data();
        dataToStore = userData;
      }
      authStore.update((curr) => {
        return {
          ...curr,
          user,
          userData: dataToStore,
          loading: false,
        };
      });
    });
  });
</script>

<div class="mainContainer">
  <slot />
</div>

<style>
  .mainContainer {
    height: 100vh;
    background: linear-gradient(to right, #000428, #000046);
    color: white;
    position: relative;
    display: flex;
    flex-direction: column;
  }
</style>
