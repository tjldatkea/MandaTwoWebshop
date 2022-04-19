<script>
	import { quintOut } from "svelte/easing";
	import { crossfade } from "svelte/transition";
	import { flip } from "svelte/animate";

	const [send, receive] = crossfade({
		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === "none" ? "" : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: (t) => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`,
			};
		},
	});

	let openModalId = "";

	let users = [
		{
			id: 0,
			name: "tjldatkea",
			description: "User written description",
			links: [
				{
					linkName: "Github",
					link: "https://github.com/anderslatif/Kea_Node_2022_Spring",
				},
				{ linkName: "Webpage", link: "https://w3spaces..." },
			],
			topic: "TopicOne - Denne og de næste skal være noget andet",
			done: false,
			group: 0,
			money: 100,
		},
		{
			id: 1,
			name: "P",
			description: "User written description",
			links: [
				{
					linkName: "Github",
					link: "https://github.com/anderslatif/Kea_Node_2022_Spring",
				},
				{ linkName: "Webpage", link: "https://w3spaces..." },
			],
			topic: "TopicOne - Denne og de næste skal være noget andet",
			done: false,
			group: 0,
			money: 100,
		},
	];

	let loggedInUserIndex = 0;

	let todos = [
		{
			id: 0,
			name: "thingOne",
			description: "kbdjvkjbsd vjvsdvjkskvds svvnsjvnk dvslsv dvsvd",
			links: [
				{
					linkName: "Github",
					link: "https://github.com/anderslatif/Kea_Node_2022_Spring",
				},
				{ linkName: "Webpage", link: "https://w3spaces..." },
			],
			topic: "TopicOne",
			done: false,
			group: 0,
			price: 5,
			madeBy: "tjldatkea",
			screenshots: ["favicon.png"],
		},
		{
			id: 1,
			name: "thingTwo",
			description: "kdvkmvdk",
			links: [
				{
					linkName: "Github",
					link: "https://github.com/anderslatif/Kea_Node_2022_Spring",
				},
				{ linkName: "Webpage", link: "https://w3spaces..." },
			],
			topic: "TopicOne",
			done: false,
			group: 0,
			price: 25,
			madeBy: "tjldatkea",
			screenshots: ["favicon.png"],
		},
		{
			id: 2,
			name: "thingThree",
			description: "description: 33333333",
			links: [
				{
					linkName: "Github",
					link: "https://github.com/anderslatif/Kea_Node_2022_Spring",
				},
				{ linkName: "Webpage", link: "https://w3spaces..." },
			],
			topic: "TopicTwo",
			done: false,
			group: 0,
			price: 45,
			madeBy: "tjldatkea",
			screenshots: ["favicon.png"],
		},
	];

	// tilføj ur og todo list, breakout og Won't Stop og MineStryger
	// progLangAndFramAndRuntimeEnv: "Made with HTML, CSS and Javascript" Kan måske bruge iconer for de forskellige sprog
	// https://devicon.dev/

	// madeBy can sættes sammen med en bruger med det navn, så man kan se brugeren der har lavet programmets profil
	// Price, discount, boughtFor, noInterestInThisProduct
	// Brugeren har sin version af virksomhedens liste over produkter og alt efter hvilken gruppe(nummer) de enkelte objekter i listen har, så ejer brugeren eller har produktet på ønskeseddel eller også er produktet synligt længst til venstre, medmindre brugeren har valgt ikke at ville se det længere. Derudover skal brugeren have nogle penge, som kan bruges til at købe produkter for.
	// hvis man er logget ind som admin, så ville det give mening at alt kan redigeres som tekstbokse eller input felter(det kan bare komme op i de alm inputfelter).

	//console.log(itemList[0].links[0].link);

	// For at sikre at der ikke er overlap mellem id'er og at man ikke får index fejl ved en tom todos liste:
	let uid =
		todos.length > 0
			? Math.max(todos.length, todos[todos.length - 1].id + 1)
			: todos.length;

	function add(input) {
		uid =
			todos.length > 0
				? Math.max(todos.length, todos[todos.length - 1].id + 1)
				: todos.length;

		const todo = {
			id: uid,
			done: false,
			name: inputName.value,
			group: Number(inputGroup.value),
			description: inputDescription.value,
			price: inputPrice.value,
			madeBy: inputMadeBy.value,
			links: [
				{
					linkName: "Github",
					link: inputGitHubLink.value,
				},
				{ linkName: "Webpage", link: inputWebpageLink.value },
			],
			screenshots: ["favicon.png"],
		};

		todos = [todo, ...todos];
		input.value = "";
		inputName.value = "";
		inputGroup.value = "";
		inputDescription.value = "";
		inputGitHubLink.value = "";
		inputWebpageLink.value = "";
		inputPrice.value = "";
		inputMadeBy.value = "";

		save();
	}

	function remove(todo) {
		todos = todos.filter((t) => t !== todo);

		// save(); // først når programmet er færdigt
	}

	// Bug: Bemærk at hvis man trykker hurtigt to gange på venstre eller højre knappen
	// på en af dem i midten, så bliver gruppe nummeret -1 eller 3 og den forsvinder
	// fra skærmen
	function changeGroupUp(todo) {
		todo.group += 1;
		todos = todos; // nødvendigt for reactivity

		// midl i forbindelse med køb(gruppe 1 -> 2)
		if (todo.group === 2) {
			users[loggedInUserIndex].money =
				users[loggedInUserIndex].money - todo.price;
		}

		if (users[loggedInUserIndex].money < 0) {
			document.getElementById("broke").style.display = "block";
			openModalId = "Broke"; //giver det her problemer hvis en anden modal er åben???
		}

		save();
	}

	function changeGroupDown(todo) {
		todo.group -= 1;
		todos = todos;

		// midl i forbindelse med køb(gruppe 1 -> 2)
		if (todo.group === 1) {
			users[loggedInUserIndex].money =
				users[loggedInUserIndex].money + todo.price; // skal være paid price, så der tages højde for en discount
		}

		save();
	}

	function addMoney() {
		users[loggedInUserIndex].money = users[loggedInUserIndex].money + 100;
		document.getElementById("broke").style.display = "none";
		openModalId = "";
	}

	//let src = '../public/image.gif';
	let src = "./image.gif"; // Andet skal der ikke stå hvis det ligger i public mappen
	// med src kunne jeg lave en modal, hvor indholdet ændres alt efter hvilken modal fra en modal liste der skal bruges, modal objektet i listen kan så have en src
	let srcEmptyWallet = "./images/emptyWallet.png";

	// function to make the topic list that is used for the menu
	function makeTopicList() {
		let topicList = ["Home"];

		for (let i = 0; i < todos.length; i++) {
			let topic = todos[i].topic;

			if (!topicList.includes(topic)) {
				topicList = [...topicList, topic];
			}
		}

		topicList = [...topicList, "login"];

		return topicList;
	}

	let menuList = makeTopicList();

	function save() {
		let listeSomJSONString = JSON.stringify(todos);

		// Gem JSON'en
		localStorage.setItem("WebshopJSON", listeSomJSONString);

		todos.forEach((element) => {
			element.saved = true;
		});
	}

	function load() {
		// Hent JSON'en
		let listeHentesSomTekst = localStorage.getItem("WebshopJSON");

		// Lav om til JSON objekt
		let listeSomJSONObjekt = JSON.parse(listeHentesSomTekst);

		todos = listeSomJSONObjekt;
		console.log("loaded");
	}

	let detailsTodo = { screenshots: ["favicon.png"], description: "" }; // måske kan det første object i todos listen bare indlæses

	// Hvis alt slettes fra listen med todos/items og programmet genstartes og detailsTodo sættes til første objekt i todoslisten, så giver det måske ikke problemer til at starte med, da man ikke får index fejl pga af todos listen i starten, men når load() er kørt, så vil der opstå problemer med undefined og andet i svelte delen med modals

	function showDetailsModal(todo) {
		closeOpenModal;

		detailsTodo = todo;

		document.getElementById("details").style.display = "block";
		openModalId = "details";
	}

	function findUser(user) {
		return user.name === selectedUserName;
	}

	let selectedUserName = "";
	let foundUser = {};

	// userName er det der kommer som argument i metodekaldet herunder fra brugerens klik
	//
	function showUserProfileModal(userName) {
		closeOpenModal();

		selectedUserName = userName; // selectedUserName er kun nødvendig hvis der bruges seperat(findUser) funktion

		foundUser = users.find(findUser);

		document.getElementById("userProfileModal").style.display = "block";
		openModalId = "userProfileModal";
	}

	function showLoginModal(selectedMenuPoint) {
		closeOpenModal();

		document.getElementById("loginModal").style.display = "block";
		openModalId = "loginModal";
	}

	function closeOpenModal() {
		console.log("openModalId: " + openModalId);
		if (openModalId !== "") {
			document.getElementById(openModalId).style.display = "none";
			openModalId = "";
		}
	}

	function keyDownHandler(e) {
		if (e.key == "Escape") {
			closeOpenModal();
			console.log("Escape trykket"); // test
		}
	}

	document.addEventListener("keydown", keyDownHandler, false);

	let grouplist = [
		{ id: 0, name: "left", title: "Shop" },
		{ id: 1, name: "middle", title: "Wishlist" },
		{ id: 2, name: "right", title: "My Precious" },
	];

	const progLangAndFramAndRuntimeEnvList = [
		{
			id: 0,
			name: "html",
			source: "https://cdn.jsdelivr.net/gh/devicons/devicon/icons/html5/html5-original.svg",
		},
		{
			id: 1,
			name: "css",
			source: "https://cdn.jsdelivr.net/gh/devicons/devicon/icons/css3/css3-original.svg",
		},
		{
			id: 2,
			name: "javascript",
			source: "https://cdn.jsdelivr.net/gh/devicons/devicon/icons/javascript/javascript-original.svg",
		},
		{
			id: 3,
			name: "svelte",
			source: "https://cdn.jsdelivr.net/gh/devicons/devicon/icons/svelte/svelte-original.svg",
		},
		{
			id: 4,
			name: "nodejs",
			source: "https://cdn.jsdelivr.net/gh/devicons/devicon/icons/nodejs/nodejs-original-wordmark.svg",
		},
		{
			id: 5,
			name: "express",
			source: "https://cdn.jsdelivr.net/gh/devicons/devicon/icons/express/express-original-wordmark.svg",
		},
	];

	// Indlæsning af items/todos ved opstart
	// Ved ændringer af objekterne der ligger i listen til at starte med skal denne udkommenteres
	//load();
</script>

<!-- <button on:click={() => save()}>Save items</button>
<button on:click={() => load()}>Load items</button> -->

{#each menuList as menuPoint}
	<ul>
		<li>
			<a href="#login">{menuPoint}</a>
			<button on:click={() => showLoginModal({ menuPoint })}>
				{menuPoint}
			</button>
		</li>
	</ul>
{/each}

<div class="board">
	<!-- Det følgende skriger på en for each, men måske skal nogle af inputene laves lidt om -->
	<input
		id="inputName"
		class="new-todo"
		placeholder="name"
		on:keydown={(event) => event.key === "Enter" && add(event.target)}
	/>
	<input
		id="inputDescription"
		class="new-todo"
		placeholder="description"
		on:keydown={(event) => event.key === "Enter" && add(event.target)}
	/>
	<input
		id="inputGroup"
		class="new-todo"
		placeholder="group (0 or 1 or 2)"
		on:keydown={(event) => event.key === "Enter" && add(event.target)}
	/>
	<input
		id="inputGitHubLink"
		class="new-todo"
		placeholder="GitHub Link"
		on:keydown={(event) => event.key === "Enter" && add(event.target)}
	/>
	<input
		id="inputWebpageLink"
		class="new-todo"
		placeholder="Webpage Link"
		on:keydown={(event) => event.key === "Enter" && add(event.target)}
	/>
	<input
		id="inputPrice"
		class="new-todo"
		placeholder="Price"
		on:keydown={(event) => event.key === "Enter" && add(event.target)}
	/>
	<input
		id="inputMadeBy"
		class="new-todo"
		placeholder="Made by"
		on:keydown={(event) => event.key === "Enter" && add(event.target)}
	/>

	<p>
		Logged in users name: {users[loggedInUserIndex].name}. User money: {users[
			loggedInUserIndex
		].money} $
	</p>

	{#each grouplist as group (group.id)}
		<div class={group.name}>
			<center><h2>{group.title}</h2></center>
			{#each todos.filter((t) => t.group == group.id) as todo (todo.id)}
				<label
					in:receive={{ key: todo.id }}
					out:send={{ key: todo.id }}
					animate:flip
				>
					<center>
						<h3>{todo.name}</h3>
						<p>{todo.description}</p>

						{#if todo.group === 2}
							{#each todo.links as lnk}
								<a href={lnk.link}>{lnk.linkName}</a><br /><br
								/>
							{/each}
							<p>Refund value: {todo.price} $</p>
						{:else}
							<p>Price: {todo.price} $</p>
						{/if}
					</center>
					{#if todo.group > 0}
						<button on:click={() => changeGroupDown(todo)}
							>left</button
						>
					{/if}
					{#if todo.group < 2}
						<button on:click={() => changeGroupUp(todo)}
							>right</button
						>
					{/if}
					<button class="closebutton" on:click={() => remove(todo)}
						>x</button
					>
					<button on:click={() => showDetailsModal(todo)}
						>details</button
					>
					<center>
						<button
							class="madeByButton"
							on:click={() => showUserProfileModal(todo.madeBy)}
						>
							Made by: {todo.madeBy} (link til profil)
						</button>
					</center>
				</label>
			{/each}
		</div>
	{/each}
</div>

<!-- Details modal with descriptions and screenshots -->
<div id="details" class="modal">
	<div class="modal-content animate">
		<div class="imgcontainer">
			<span
				onclick="document.getElementById('details').style.display='none'"
				class="close"
				title="Close Modal">&times;</span
			>
			<h2>
				{detailsTodo.name}
			</h2>
		</div>
		<div>
			<center>
				<p>{detailsTodo.description}</p>
			</center>
		</div>

		<div>
			<center>
				{#each detailsTodo.screenshots as sc}
					<img
						src="./images/{sc}"
						alt="Screenshot: {sc}"
						class="screenshot"
					/>
				{/each}
			</center>
		</div>

		<!-- Evt kan der også være links her, hvis det er gruppe 2
		Er der andet der giver mening at det er her under details???
		-->

		<div>
			<center>
				<!-- Kunne være fint med en ellipse rundt om navnet eller noget andet,
				jeg burde bare kunne lave en knap der har den form -->
				<button
					class="madeByButton"
					on:click={() => showUserProfileModal(detailsTodo.madeBy)}
				>
					Made by: {detailsTodo.madeBy} (link til profil)
				</button>

				{#each progLangAndFramAndRuntimeEnvList as entity (entity.id)}
					{#if detailsTodo.description
						.toLowerCase()
						.includes(entity.name)}
						<img
							class="icons"
							src={entity.source}
							alt={entity.name}
						/>
					{/if}
				{/each}
			</center>
		</div>

		<div class="container" style="background-color:#f1f1f1">
			<button
				type="button"
				onclick="document.getElementById('details').style.display='none'"
				class="cancelbtn">Close</button
			>
		</div>
	</div>
</div>

<!-- login modal -->
<div id="loginModal" class="modal">
	<form class="modal-content animate" action="/action_page.php" method="post">
		<div class="imgcontainer">
			<span
				onclick="document.getElementById('login').style.display='none'"
				class="close"
				title="Close Modal">&times;</span
			>
			<img {src} class="avatar" alt="man dancing" />
		</div>

		<div class="container">
			<label for="uname"><b>Username</b></label>
			<input
				type="text"
				placeholder="Enter Username"
				name="uname"
				required
			/>

			<label for="psw"><b>Password</b></label>
			<input
				type="password"
				placeholder="Enter Password"
				name="psw"
				required
			/>

			<button type="submit">Login</button>
			<label>
				<input type="checkbox" checked="checked" name="remember" /> Remember
				me
			</label>
		</div>

		<div class="container" style="background-color:#f1f1f1">
			<button
				type="button"
				onclick="document.getElementById('login').style.display='none'"
				class="cancelbtn">Cancel</button
			>
			<span class="psw">Forgot <a href="#">password?</a></span>
		</div>
	</form>
</div>

<!-- user profile modal -->
<div id="userProfileModal" class="modal">
	<form class="modal-content animate" action="/action_page.php" method="post">
		<div class="imgcontainer">
			<span
				onclick="document.getElementById('userProfileModal').style.display='none'"
				class="close"
				title="Close Modal">&times;</span
			>
			<img src="./images/avatar.png" alt="Avatar" class="avatar" />
		</div>

		<div class="container">
			<center>
				<h2><b>{foundUser.name}</b></h2>
			</center>
		</div>

		{#each todos.filter((t) => t.madeBy === foundUser.name) as todo (todo.id)}
			<button type="button" on:click={() => showDetailsModal(todo)}
				>{todo.name}
			</button>
		{/each}

		<!-- Evt brugerens progSprog mm her  -->

		<div class="container" style="background-color:#f1f1f1">
			<button
				type="button"
				onclick="document.getElementById('userProfileModal').style.display='none'"
				class="cancelbtn">Close</button
			>
		</div>
	</form>
</div>

<!-- Broke modal -->
<div id="broke" class="modal">
	<!--  ********* HERTIL ********
		Giver problemer når denne udkommenteres 
		<form class="modal-content animate" action="/action_page.php" method="post"> -->
	<div class="modal-content animate">
		<div class="imgcontainer">
			<span
				onclick="document.getElementById('broke').style.display='none'"
				class="close"
				title="Close Modal">&times;</span
			>

			<img src={srcEmptyWallet} alt="empty wallet with moth flying" />
		</div>

		<button on:click={() => addMoney()}>Add Money</button>
	</div>
</div>

<style>
	.new-todo {
		font-size: 1em;
		width: 100%;
		margin: 1em 0 1em 0;
	}

	.board {
		max-width: 100%; /*36em;*/
		margin: 0 auto;
	}

	.left,
	.middle,
	.right {
		float: left;
		width: 33%; /* 100 / {numberOfGroups}; det her virker ikke, men man kan istedet bare lave forskellige klasser med 100% 50% 33% og evt 25% og skifte klasse navnet i html'en og på den måde ændre bredden på dem og man evt også bare gøre det inline, så ville man måske bare kunne skrive 100 / {numberOfGroups};*/
		padding: 0 1em 0 0;
		box-sizing: border-box;
	}

	h2 {
		font-size: 2em;
		font-weight: 200;
		user-select: none;
	}

	label {
		top: 0;
		left: 0;
		display: block;
		font-size: 1em;
		line-height: 1;
		padding: 0.5em;
		margin: 0 auto 0.5em auto;
		border-radius: 2px;
		background-color: #eee;
		user-select: none;
	}

	input {
		margin: 0;
	}

	.right label {
		background-color: rgb(180, 240, 100);
	}

	.left label {
		background-color: red; /*rgb(180,240,100);*/
	}

	.middle label {
		background-color: orange; /*rgb(180,240,100);*/
	}

	.closebutton {
		float: right;
		height: 1em;
		box-sizing: border-box;
		padding: 0 0.5em;
		line-height: 1;
		background-color: transparent;
		border: none;
		color: rgb(170, 30, 30);
		opacity: 0;
		transition: opacity 0.2s;
	}

	/* Jeg har brug for en label for at kunne animere at elementerne flytter sig, men
	hvis jeg udkommentere denne for at alle knapperne opfører sig ens mht 'opacity', så 
	sker der noget underligt med knapperne, når man 'hover' over dem. Indtil problemet 
	er løst er denne herunder nødvendig */
	label:hover button {
		opacity: 1;
	}

	body {
		font-family: Arial, Helvetica, sans-serif;
	}

	/* Full-width input fields */
	input[type="text"],
	input[type="password"] {
		width: 100%;
		padding: 12px 20px;
		margin: 8px 0;
		display: inline-block;
		border: 1px solid #ccc;
		box-sizing: border-box;
	}

	/* Set a style for all buttons */
	button {
		background-color: #04aa6d;
		color: white;
		padding: 14px 20px;
		margin: 8px 0;
		border: none;
		cursor: pointer;
		width: 100%;
	}

	button:hover {
		opacity: 0.9;
	}

	/* Extra styles for the cancel button */
	.cancelbtn {
		width: auto;
		padding: 10px 18px;
		background-color: #f44336;
	}

	.madeByButton {
		width: 100%;
		padding: 12px 20px;
		margin: 8px 0;
		display: inline-block;
		border: 1px solid #ccc;
		box-sizing: border-box;
		background-color: transparent;
		color: black;
	}

	/* 	
	.cancelbtn:hover,
	.cancelbtn:focus {
		color: black;
		cursor: pointer;
		width: 100%;
	}
	*/

	/* Center the image and position the close button */
	.imgcontainer {
		text-align: center;
		margin: 24px 0 12px 0;
		position: relative;
	}

	/*
	img {
		width: 40%;
		border-radius: 50%;
	}
*/
	img.avatar {
		width: 40%;
		border-radius: 50%;
	}

	img.icons {
		width: 7%;
	}

	img.screenshot {
		width: 90%;
		/*height:fit-content;*/
	}

	/*
	.img.screenshot:hover,
	.img.screenshot:focus {
		color: red;
		cursor: pointer;
		width: 100%;
	}
*/

	.container {
		padding: 16px;
	}

	span.psw {
		float: right;
		padding-top: 16px;
	}

	/* The Modal (background) */
	.modal {
		display: none; /* Hidden by default */
		position: fixed; /* Stay in place */
		z-index: 1; /* Sit on top */
		left: 0;
		top: 0;
		width: 100%; /* Full width */
		height: 100%; /* Full height */
		overflow: auto; /* Enable scroll if needed */
		background-color: rgb(0, 0, 0); /* Fallback color */
		background-color: rgba(0, 0, 0, 0.4); /* Black w/ opacity */
		padding-top: 60px;
	}

	/* Modal Content/Box */
	.modal-content {
		background-color: #fefefe;
		margin: 5% auto 15% auto; /* 5% from the top, 15% from the bottom and centered */
		border: 1px solid #888;
		width: 80%; /* Could be more or less, depending on screen size */
	}

	/* The Close Button (x) */
	.close {
		position: absolute;
		right: 25px;
		top: 0;
		color: #000;
		font-size: 35px;
		font-weight: bold;
	}

	.close:hover,
	.close:focus {
		color: red;
		cursor: pointer;
	}

	/* Add Zoom Animation */
	.animate {
		-webkit-animation: animatezoom 0.6s;
		animation: animatezoom 0.6s;
	}

	@-webkit-keyframes animatezoom {
		from {
			-webkit-transform: scale(0);
		}
		to {
			-webkit-transform: scale(1);
		}
	}

	@keyframes animatezoom {
		from {
			transform: scale(0);
		}
		to {
			transform: scale(1);
		}
	}

	/* Change styles for span and cancel button on extra small screens */
	@media screen and (max-width: 300px) {
		span.psw {
			display: block;
			float: none;
		}
		.cancelbtn {
			width: 100%;
		}
	}

	ul {
		list-style-type: none;
		margin: 0;
		padding: 0;
		overflow: hidden;
		background-color: #333;
	}

	li {
		float: left;
	}

	li a {
		display: block;
		color: white;
		text-align: center;
		padding: 14px 16px;
		text-decoration: none;
	}

	li a:hover:not(.active) {
		background-color: #111;
	}

	.active {
		background-color: #04aa6d;
	}
</style>
