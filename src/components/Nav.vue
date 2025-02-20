<template>
	<header :class="{ 'scrolled-nav': scrollNav }">
		<nav>
			<div class="branding">
				<img src="../assets/BsoC logo.svg" alt="img" />
			</div>

			<!-- Navbar -->
			<div class="nav-div">
				<ul v-show="!mobile" class="navigations">
					
					<li>
						<router-link
							class="link"
							to="/home"
							@click.native="setActiveMenuItemAndCloseNav('home')"
						>
							Home
						</router-link>
					</li>

					<li>
						<router-link
							class="link"
							to="/dashboard"
							@click.native="setActiveMenuItemAndCloseNav('dashboard')"
						>
							Dashboard
						</router-link>
					</li>

					<li>
						<router-link
							class="link"
							to="/projects"
							@click.native="setActiveMenuItemAndCloseNav('projects')"
						>
							Projects
						</router-link>
					</li>

					<template v-if="isLoggedIn">
						<li>
							<router-link
								class="link"
								to="/submit"
								@click.native="setActiveMenuItemAndCloseNav('submit')"
							>
								Submit PR
							</router-link>
						</li>

						<li>
							<router-link
								class="link"
								to="/myPR"
								@click.native="setActiveMenuItemAndCloseNav('myPR')"
							>
								My PR
							</router-link>
						</li>

                    <div class="line"></div>
					<div class="animation start-home">
						<div class="one"></div>
                        <div class="two"></div>
					</div>
						
					</template>
				</ul>
			</div>

			<div class="auth-div">
				<ul v-show="!mobile" class="authnavigations">
					<template v-if="!isLoggedIn">
						<li>
							<router-link class="auth" to="/auth"> Login </router-link>
						</li>

						<li>
							<router-link class="auth" to="/signup"> Sign Up </router-link>
						</li>
					</template>

					<template v-else>
						<li @click="logout" class="auth">Log Out</li>
					</template>
				</ul>
			</div>

			<div class="icon" @click="toggleMobileNav">
				<i
					v-show="mobile && !mobileNav"
					class="fas fa-bars"
					:class="{ 'icon-active': mobileNav }"
					style="
						font-size: 1.5rem;
						position: fixed;
						top: 20px;
						right: 25px;
						cursor: pointer;
						z-index: 100;
					"
				></i>
			</div>
             

        <!-- Hamburger -->

			<div class="icon" @click="toggleMobileNav">
				<i
					v-show="mobile && mobileNav"
					class="fas fa-times"
					:class="{ 'icon-active': mobileNav }"
					style="
						font-size: 1.5rem;
						position: fixed;
						top: 20px;
						right: 25px;
						cursor: pointer;
						z-index: 100;
					"
				></i>
			</div>

			<transition name="mobile-nav">
				<ul v-show="mobileNav" class="dropdown-nav">
					<div class="imageham">
						<img src="../assets/BsoC logo.svg" alt="img" />
					</div>
					<li>
						<router-link
							class="link"
							:class="{ active: activeMenuItem === 'home' }"
							to="/home"
							@click.native="setActiveMenuItemAndCloseNav('home')"
						>
							Home
						</router-link>
					</li>

					<li>
						<router-link
							class="link"
							:class="{ active: activeMenuItem === 'dashboard' }"
							to="/dashboard"
							@click.native="setActiveMenuItemAndCloseNav('dashboard')"
						>
							Dashboard
						</router-link>
					</li>

					<li>
						<router-link
							class="link"
							:class="{ active: activeMenuItem === 'projects' }"
							to="/projects"
							@click.native="setActiveMenuItemAndCloseNav('projects')"
						>
							Projects
						</router-link>
					</li>

					<template v-if="isLoggedIn">
						<li>
							<router-link
								class="link"
								:class="{ active: activeMenuItem === 'myPR' }"
								to="/mypr"
								@click.native="setActiveMenuItemAndCloseNav('myPR')"
							>
								My PR
							</router-link>
						</li>

						<li @click="logout" class="link">Log out</li>
					</template>

					<template v-else>
						<li>
							<router-link
								class="link"
								:class="{ active: activeMenuItem === 'signup' }"
								to="/signup"
								@click.native="setActiveMenuItemAndCloseNav('signup')"
							>
								Sign Up
							</router-link>
						</li>
						<li>
							<router-link
								class="link"
								:class="{ active: activeMenuItem === 'auth' }"
								to="/login"
								@click.native="toggleLogin()"
							>
								Login
							</router-link>
						</li>
					</template>
				</ul>
			</transition>
		</nav>
	</header>
	<Login
		v-if="showLogin"
		:toggle="toggleLogin"
		@loginSuccess="handleLoginSuccess"
	/>
	<Signup
		v-if="showSignup"
		:toggle="toggleSignup"
		@signupSuccess="handleSignupSuccess"
	/>
</template>

<script>
import { projectAuth } from '../firebase/config'
import Login from '@/components/Login'
import Signup from '@/components/Signup'
import { successToast, errorToast } from '../composables/useToast'

export default {
	name: 'Nav',
	components: {
		Login,
		Signup,
	},
	data() {
		return {
			scrollNav: false,
			mobile: false,
			mobileNav: false,
			windowWidth: null,
			isLoggedIn: false,
			showLogin: false,
			showSignup: false,
			activeMenuItem: '',
		}
	},
	created() {
		window.addEventListener('resize', this.checkScreen)
		this.checkScreen()
		this.checkUserLoggedIn()
	},
	mounted() {
		window.addEventListener('scroll', this.updateScroll)
	},
	methods: {
		toggleMobileNav() {
			this.mobileNav = !this.mobileNav
		},
		closeMobileNav() {
			this.mobileNav = false
		},
		toggleLogin() {
			this.showLogin = !this.showLogin
		},
		toggleSignup() {
			this.showSignup = !this.showSignup
		},
		checkScreen() {
			this.windowWidth = window.innerWidth
			if (this.windowWidth <= 850) {
				this.mobile = true
			} else {
				this.mobile = false
				this.mobileNav = false
			}
		},
		updateScroll() {
			const scrollPosition = window.scrollY
			this.scrollNav = scrollPosition > 50
		},
		async checkUserLoggedIn() {
			projectAuth.onAuthStateChanged((user) => {
				this.isLoggedIn = !!user
			})
		},
		async logout() {
			try {
				await projectAuth.signOut()
				this.isLoggedIn = false
				successToast('Success', 'Logout successful')
			} catch (error) {
				errorToast('Error', 'Logout failed')
				console.error('Logout error:', error.message)
			}
		},
		setActiveMenuItemAndCloseNav(item) {
			this.activeMenuItem = item
			this.mobileNav = false
		},
		handleLoginSuccess() {
			successToast('Success', 'Login successful')
			this.showLogin = false
		},
		handleSignupSuccess() {
			successToast('Success', 'Signup successful')
			this.showSignup = false
		},
	},
}
</script>

<style scoped>
header {
	z-index: 99;
	position: fixed;
	width: 100%;
	transition: 0.5s ease all;
	color: white;
}

nav {
	display: flex;
	width: 100%;
	box-sizing: border-box;
	margin-top: 0.6%;

	ul,
	.link {
		font-weight: 500;
		color: white;
		list-style: none;
		align-items: center;
		text-align: center;
		text-decoration: none;
	}

	li {
		padding: 0.9%;
        margin-left: 4%;
		width: 50%;
		z-index: 100;
		margin-top: 3%;
	}

	.link {
		font-size: 130%;
		transition: 0.5s ease all;
		padding-bottom: 0.2%;
		border-bottom: 0.1rem solid transparent;
		cursor: pointer;
		
		text-align: center;

		&:hover {
			color: black;
			z-index: 100;
		}
	}

	.branding {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 10%;
		margin-top: 1.3%;

		img {
			width: 40%;
			transition: 0.5s ease all;
		}
	}

	.imageham {
		
		position: relative;
		top:-40px;
		left:-110px;
		cursor: pointer;
		
		img {
			width: 50%;
			transition: 0.5s ease all;
		}
	}

	.nav-div {
		position: relative;
		width: 70%;
		padding-left: 4%;
		justify-content: center;
		align-items: center;
	}

	.navigations {
		position: relative;
		display: flex;
		justify-content: center;
		align-items: center;
		list-style-type: none;
	}

	.navigations .animation {
		position: fixed;
		height: 10%;
		top: 20px;
		z-index: 0;
		text-align: center;
		justify-content: center;
		background-color: #0891b3;
		border-radius: 0 0 40px 40px;
		transition: all 0.5s ease 0s;
	}
	
	.navigations .animation .one {
		position: relative;
		height: 150%;
		width:150%;
		border-radius: 1000px;
		top:-5px;
		left:81%;
		content: '';
		z-index:-1;
		border-left: 20px solid #0891b3;
		border-right: 20px solid transparent;
		border-top: 20px solid transparent;
		border-bottom: 20px solid transparent;
		transform:rotate(30deg);
	}

	.navigations .animation .two{
		position: relative;
		height: 150%;
		width:150%;
		border-radius: 1000px;
		top:-125px;
		right:130.5%;
		content: '';
		z-index:-1;

		border-left: 20px solid transparent;
		border-right: 20px solid #0891b3;
		border-top: 20px solid transparent;
		border-bottom: 20px solid transparent;
		transform:rotate(-30deg);
	}
	
	

	
	.navigations li:nth-child(1) {
		width: 13%;
	}
	.navigations .start-home,
	li:nth-child(1):hover ~ .animation {
		width: 9%;
		left: 23.8%;
	}
	.navigations li:nth-child(2) {
		width: 12%;
	}
	.navigations li:nth-child(2):hover ~ .animation {
		width: 9%;
		left: 34.7%;
	}
	.navigations li:nth-child(3) {
		width: 13.7%;
	}
	.navigations li:nth-child(3):hover ~ .animation {
		width: 9%;
		left: 45.2%;
	}
	.navigations li:nth-child(4) {
		width: 13%;
	}
	.navigations li:nth-child(4):hover ~ .animation {
		width: 9%;
		left: 56.2%;
	}
	.navigations li:nth-child(5) {
		width: 11%;
	}
	.navigations li:nth-child(5):hover ~ .animation {
		width: 9%;
		left: 66.4%;
	}

	.auth-div {
		width: 20%;
		padding-left: 3rem;
		top:10%
	}
	.authnavigations {
		display: flex;
	}

	.icon {
		position: absolute;
		left: 50px;
		display: flex;
		align-items: center;
		height: 100%;
		cursor: pointer;
		font-size: 1.5rem;
		transition: 0.5s ease all;
	}

	.icon-active {
		transform: rotate(180deg);
	}

	.dropdown-nav {
		display: flex;
		flex-direction: column;
		position: fixed;
		width: 100%;
		max-width: 15.62rem;
		height: 100%;
		background-color: #18181b;
		top: 0;
		right: 0;
		padding-top: 3.75rem;
	}

	.dropdown-nav .link.active {
		display: block;
		background-color: #0891b3;

		border-radius: 1.25rem;

		text-align: center;
		color: white;
		height: 2.68rem;
		width: 100%;
		transition: background-color 0.7s ease;
	}

	.dropdown-nav .link.active:hover {
		background-color: #0891b3;
	}

	.mobile-nav-enter-active,
	.mobile-nav-leave-active {
		transition: 1s ease all;
	}

	.mobile-nav-enter-from,
	.mobile-nav-leave-to {
		transform: translateX(15.625rem);
	}

	.mobile-nav-enter-to {
		transform: translateX(0);
	}

	.auth {
		color: #0891b3;
		border: #0891b3 1px solid;
		font-size: 140%;
		border-radius: 6rem;
		cursor: pointer;
		text-align: center;
		text-decoration: none;
		margin-top: 15%;
		&:hover {
			color: white;
		}
	}

	.scrolled-nav {
		background-color: black;
		box-shadow:
			0 4px 6px -1px rgba(0, 0, 0, 0.1),
			0 2px 4px -1px rgba(0, 0, 0, 0.6);

		> nav {
			padding: 8px 5%;

			.branding {
				img {
					width: 40px;
					box-shadow:
						0 4px 6px -1px rgba(0, 0, 0, 0.1),
						0 2px 4px -1px rgba(0, 0, 0, 0.6);
				}
			}
		}
	}
	.line{
		position: fixed;
		background-color: #0891b3;
		width: 200vw;
		height: 20px;
		top:0;
	}

	@media (max-width: 1374px) {
		.navigations .animation {
			position: fixed;
			height: 6%;
			top: 2%;
			z-index: 0;
			background-color: #0891b3;
			border-radius: 0.5rem;
			transition: all 0.5s ease 0s;
		}
	  }

	  @media (max-width: 1243px) {

		.auth {
			color: #0891b3;
			border: #0891b3 1px solid;
			font-size: 110%;
			border-radius: 6rem;
			cursor: pointer;
			text-align: center;
			text-decoration: none;
			margin-top: 8.5%;
			&:hover {
				color: white;
			}
		}

        li {
			padding: 0.2%;
			font-size: 80%;
			margin-left: 4%;
			top:10%;
			width: 60%;
			z-index: 100;
			margin-top:2%;
		}

		.navigations .animation {
			position: fixed;
			height: 5%;
			top: 2.3%;
			z-index: 0;
			background-color: #0891b3;
			border-radius: 0.5rem;
			transition: all 0.5s ease 0s;
		}

        .navigations .animation {
			position: fixed;
			height: 10%;
			top: 20px;
			z-index: 0;
			text-align: center;
			justify-content: center;
			background-color: #0891b3;
			border-radius: 0 0 40px 40px;
			transition: all 0.5s ease 0s;
		}
		
		.navigations .animation .one {
			position: relative;
			height: 150%;
			width:150%;
			border-radius: 1000px;
			top:-5px;
			left:81%;
			content: '';
			z-index:-1;
			border-left: 20px solid #0891b3;
			border-right: 20px solid transparent;
			border-top: 20px solid transparent;
			border-bottom: 20px solid transparent;
			transform:rotate(30deg);
		}

			.navigations li:nth-child(1) {
				width: 12%;
			}
			.navigations .start-home,
			li:nth-child(1):hover ~ .animation {
				width: 8%;
				left:24.4%;
			}
			.navigations li:nth-child(2) {
				width: 13%;
			}
			.navigations li:nth-child(2):hover ~ .animation {
				width: 10%;
				left: 34%;
			}
			.navigations li:nth-child(3) {
				width: 13.7%;
			}
			.navigations li:nth-child(3):hover ~ .animation {
				width: 7%;
				left: 46.2%;
			}
			.navigations li:nth-child(4) {
				width: 13%;
			}
			.navigations li:nth-child(4):hover ~ .animation {
				width: 9%;
				left: 56.2%;
			}
			.navigations li:nth-child(5) {
				width: 11%;
			}
			.navigations li:nth-child(5):hover ~ .animation {
				width: 7%;
				left: 67.4%;
			}
		}

		@media (max-width: 991px) {

			.auth {
				color: #0891b3;
				border: #0891b3 1px solid;
				font-size: 80%;
				border-radius: 6rem;
				cursor: pointer;
				text-align: center;
				text-decoration: none;
				margin-top: 10%;
				&:hover {
					color: white;
				}
			}
	
			li {
				font-size: 67%;
				margin-left: 3%;
				top:10%;
				width: 60%;
				z-index: 100;
				margin-top:2%;
			}
	
			.navigations .animation {
				position: fixed;
				height: 4%;
				top: 2%;
				z-index: 0;
				background-color: #0891b3;
				border-radius: 0.5rem;
				transition: all 0.5s ease 0s;
			}
				.navigations li:nth-child(1) {
					width: 12%;
				}
				.navigations .start-home,
				li:nth-child(1):hover ~ .animation {
					width: 7%;
					left:26.7%;
				}
				.navigations li:nth-child(2) {
					width: 13%;
				}
				.navigations li:nth-child(2):hover ~ .animation {
					width: 10%;
					left: 35%;
				}
				.navigations li:nth-child(3) {
					width: 13.7%;
				}
				.navigations li:nth-child(3):hover ~ .animation {
					width: 7%;
					left: 46.4%;
				}
				.navigations li:nth-child(4) {
					width: 13%;
				}
				.navigations li:nth-child(4):hover ~ .animation {
					width: 9%;
					left: 55.8%;
				}
				.navigations li:nth-child(5) {
					width: 11%;
				}
				.navigations li:nth-child(5):hover ~ .animation {
					width: 7%;
					left: 66%;
				}
			}
	}

	 
	
  
</style>
