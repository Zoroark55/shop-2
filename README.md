<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GameAccount Marketplace</title>
    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-auth.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>
    <style>
        /* Previous CSS remains, add these new styles */
        .auth-wall {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.9);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 10000;
        }

        .sell-section {
            display: none;
            padding: 2rem;
            margin-top: 80px;
        }

        .upload-area {
            border: 2px dashed #6c5ce7;
            padding: 1rem;
            margin: 1rem 0;
            text-align: center;
        }

        .preview-images {
            display: grid;
            grid-template-columns: repeat(5, 1fr);
            gap: 0.5rem;
            margin: 1rem 0;
        }

        .preview-img {
            width: 100%;
            height: 100px;
            object-fit: cover;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <!-- Auth Wall -->
    <div class="auth-wall" id="authWall">
        <div class="auth-box">
            <h2>Welcome to GameAccount Marketplace</h2>
            <button id="googleSignIn">Sign In with Google</button>
        </div>
    </div>

    <!-- Main Content (hidden until auth) -->
    <div id="mainContent" style="display: none;">
        <!-- Header and previous sections remain -->

        <!-- Sell Section -->
        <section class="sell-section" id="sellSection">
            <h2>Sell Your Game Account</h2>
            <form id="sellForm">
                <div class="upload-area">
                    <input type="file" id="coverPhoto" accept="image/*" required>
                    <label for="coverPhoto">Upload Cover Photo*</label>
                </div>

                <div class="upload-area">
                    <input type="file" id="additionalPhotos" accept="image/*" multiple>
                    <label for="additionalPhotos">Upload Additional Photos (max 4)</label>
                </div>

                <div class="preview-images" id="imagePreview"></div>

                <input type="email" id="accountEmail" placeholder="Account Email" required>
                <input type="password" id="accountPassword" placeholder="Account Password" required>
                <input type="number" id="accountPrice" placeholder="Price in Z-Points" required>
                
                <button type="submit">List Account</button>
            </form>
        </section>
    </div>

    <script>
        // Firebase Configuration
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_AUTH_DOMAIN",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_STORAGE_BUCKET",
            messagingSenderId: "YOUR_SENDER_ID",
            appId: "YOUR_APP_ID"
        };
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();

        // Auth State Listener
        auth.onAuthStateChanged(user => {
            if (user) {
                document.getElementById('authWall').style.display = 'none';
                document.getElementById('mainContent').style.display = 'block';
                loadListings();
            } else {
                document.getElementById('authWall').style.display = 'flex';
                document.getElementById('mainContent').style.display = 'none';
            }
        });

        // Google Sign-In
        document.getElementById('googleSignIn').addEventListener('click', () => {
            const provider = new firebase.auth.GoogleAuthProvider();
            auth.signInWithPopup(provider);
        });

        // Image Preview Handler
        document.getElementById('additionalPhotos').addEventListener('change', function(e) {
            const preview = document.getElementById('imagePreview');
            preview.innerHTML = '';
            
            Array.from(e.target.files).slice(0,4).forEach(file => {
                const reader = new FileReader();
                reader.onload = (e) => {
                    const img = document.createElement('img');
                    img.src = e.target.result;
                    img.classList.add('preview-img');
                    preview.appendChild(img);
                }
                reader.readAsDataURL(file);
            });
        });

        // Sell Form Submission
        document.getElementById('sellForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            
            // Basic encryption example (use proper encryption in production)
            const encoder = new TextEncoder();
            const emailData = encoder.encode(document.getElementById('accountEmail').value);
            const passwordData = encoder.encode(document.getElementById('accountPassword').value);
            
            const encryptedEmail = btoa(String.fromCharCode(...new Uint8Array(emailData)));
            const encryptedPassword = btoa(String.fromCharCode(...new Uint8Array(passwordData)));

            // Store in Firestore
            await db.collection('listings').add({
                email: encryptedEmail,
                password: encryptedPassword,
                price: document.getElementById('accountPrice').value,
                seller: auth.currentUser.uid,
                timestamp: firebase.firestore.FieldValue.serverTimestamp()
            });

            alert('Account listed successfully!');
            window.location.hash = '';
        });

        // Load Listings
        async function loadListings() {
            const listings = await db.collection('listings').get();
            // Update your HTML with listings data
        }
    </script>
</body>
</html>
