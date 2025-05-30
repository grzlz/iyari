<script lang="ts">
	let showPassword = false;
	let darkMode = false;
	let formErrors: Record<string, string> = {};
	let passwordStrength = '';
	let success = false;

	function evaluatePasswordStrength(password: string) {
		if (password.length < 8) return '';
		const hasUpper = /[A-Z]/.test(password);
		const hasLower = /[a-z]/.test(password);
		const hasNumber = /\d/.test(password);
		const hasSpecial = /[!@#$%^&*(),.?":{}|<>]/.test(password);

		const score = [hasUpper, hasLower, hasNumber, hasSpecial].filter(Boolean).length;

		if (score <= 1) return 'Débil';
		if (score === 2 || score === 3) return 'Media';
		if (score === 4) return 'Fuerte';
		return '';
	}

	function handlePasswordInput(event: Event) {
		const target = event.target as HTMLInputElement;
		passwordStrength = evaluatePasswordStrength(target.value);
	}

	function formatPhone(value: string) {
		const digits = value.replace(/\D/g, '').substring(0, 10);
		let formatted = digits;
		if (digits.length > 6) {
			formatted = digits.replace(/(\d{3})(\d{3})(\d{1,4})/, '$1-$2-$3');
		} else if (digits.length > 3) {
			formatted = digits.replace(/(\d{3})(\d{1,3})/, '$1-$2');
		}
		return formatted;
	}

	function handlePhoneInput(event: Event) {
		const target = event.target as HTMLInputElement;
		target.value = formatPhone(target.value);
	}

	function validateForm(formData: FormData) {
		formErrors = {};
		const requiredFields = [
			'nombre',
			'celular',
			'direccion',
			'referencia1',
			'referencia2',
			'telefono1',
			'telefono2',
			'identificacion',
			'password'
		];

		for (const field of requiredFields) {
			const value = formData.get(field);
			if (!value || (value instanceof File && value.size === 0)) {
				formErrors[field] = 'Este campo es obligatorio.';
			}
		}

		const celular = formData.get('celular') as string;
		if (celular && !/^\d{3}-\d{3}-\d{4}$/.test(celular)) {
			formErrors.celular = 'El celular debe tener formato 999-999-9999.';
		}

		const telefono1 = formData.get('telefono1') as string;
		if (telefono1 && !/^\d{3}-\d{3}-\d{4}$/.test(telefono1)) {
			formErrors.telefono1 = 'Teléfono de referencia 1 inválido.';
		}

		const telefono2 = formData.get('telefono2') as string;
		if (telefono2 && !/^\d{3}-\d{3}-\d{4}$/.test(telefono2)) {
			formErrors.telefono2 = 'Teléfono de referencia 2 inválido.';
		}

		const password = formData.get('password') as string;
		if (!password || password.length < 8 || !/[A-Z]/.test(password) || !/\d/.test(password)) {
			formErrors.password = 'Debe tener al menos 8 caracteres, una mayúscula y un número.';
		}

		return Object.keys(formErrors).length === 0;
	}

	async function handleSubmit(event: Event) {
		event.preventDefault();
		const form = event.target as HTMLFormElement;
		const formData = new FormData(form);

		// Mostrar todos los valores en consola para debug
		console.log('Formulario enviado con datos:');
		formData.forEach((value, key) => {
			if (value instanceof File) {
				console.log(`${key}: File name = ${value.name}, size = ${value.size} bytes`);
			} else {
				console.log(`${key}: ${value}`);
			}
		});

		if (validateForm(formData)) {
			success = true;
			formErrors = {};
			passwordStrength = '';
			form.reset();

			// Simular redirección a otra página después de 1.5 segundos
			setTimeout(() => {
				window.location.href = '/gracias'; // aquí ajusta la ruta real
			}, 1500);
		} else {
			success = false;
		}
	}

	function toggleDarkMode() {
		darkMode = !darkMode;
	}
</script>

<div class="mode-toggle-container">
	<button type="button" on:click={toggleDarkMode} class="mode-toggle-btn" aria-label="Alternar modo claro/oscuro">
		{darkMode ? 'Modo Claro' : 'Modo Oscuro'}
	</button>
</div>

{#if success}
	<p class="success-message">¡Formulario enviado con éxito! Redirigiendo...</p>
{/if}

<form
	class="form-container"
	on:submit={handleSubmit}
	autocomplete="off"
	class:dark-mode={darkMode}
	enctype="multipart/form-data"
>
	<h2 class="form-title">Registro como trabajador asociado - Iyari Cancún</h2>

	<div class="form-group">
		<label title="Escribe tu nombre completo">Nombre:</label>
		<input name="nombre" placeholder="Tu nombre completo" required />
		{#if formErrors.nombre}
			<p class="error-message">⚠️ {formErrors.nombre}</p>
		{/if}
	</div>

	<div class="form-group">
		<label title="Número celular con 10 dígitos, ejemplo: 999-999-9999">Celular:</label>
		<input
			name="celular"
			type="tel"
			placeholder="999-999-9999"
			on:input={handlePhoneInput}
			maxlength="12"
			required
		/>
		{#if formErrors.celular}
			<p class="error-message">⚠️ {formErrors.celular}</p>
		{/if}
	</div>

	<div class="form-group">
		<label title="Dirección completa, donde estarás disponible">Dirección:</label>
		<input name="direccion" placeholder="Tu dirección completa" required />
		{#if formErrors.direccion}
			<p class="error-message">⚠️ {formErrors.direccion}</p>
		{/if}
	</div>

	<div class="form-group">
		<label title="Nombre de tu referencia 1">Referencia 1:</label>
		<input name="referencia1" placeholder="Nombre de tu referencia 1" required />
		{#if formErrors.referencia1}
			<p class="error-message">⚠️ {formErrors.referencia1}</p>
		{/if}
	</div>

	<div class="form-group">
		<label title="Teléfono de tu referencia 1, formato 999-999-9999">Teléfono Referencia 1:</label>
		<input
			name="telefono1"
			type="tel"
			placeholder="999-999-9999"
			on:input={handlePhoneInput}
			maxlength="12"
			required
		/>
		{#if formErrors.telefono1}
			<p class="error-message">⚠️ {formErrors.telefono1}</p>
		{/if}
	</div>

	<div class="form-group">
		<label title="Nombre de tu referencia 2">Referencia 2:</label>
		<input name="referencia2" placeholder="Nombre de tu referencia 2" required />
		{#if formErrors.referencia2}
			<p class="error-message">⚠️ {formErrors.referencia2}</p>
		{/if}
	</div>

	<div class="form-group">
		<label title="Teléfono de tu referencia 2, formato 999-999-9999">Teléfono Referencia 2:</label>
		<input
			name="telefono2"
			type="tel"
			placeholder="999-999-9999"
			on:input={handlePhoneInput}
			maxlength="12"
			required
		/>
		{#if formErrors.telefono2}
			<p class="error-message">⚠️ {formErrors.telefono2}</p>
		{/if}
	</div>

	<div class="form-group">
		<label title="Sube tu identificación oficial en formato PDF o imagen">Identificación oficial (PDF o imagen):</label>
		<input type="file" name="identificacion" accept=".pdf,image/*" required />
		{#if formErrors.identificacion}
			<p class="error-message">⚠️ {formErrors.identificacion}</p>
		{/if}
	</div>

	<div class="form-group">
		<label title="Opcional: Carta de antecedentes no penales en PDF o imagen">Carta de antecedentes no penales (opcional):</label>
		<input type="file" name="carta_antecedentes" accept=".pdf,image/*" />
	</div>

	<div class="form-group password-group">
		<label title="Crea una contraseña segura">Contraseña:</label>
		<div class="password-wrapper">
			<input
				type={showPassword ? 'text' : 'password'}
				name="password"
				on:input={handlePasswordInput}
				placeholder="Mínimo 8 caracteres, 1 mayúscula, 1 número"
				required
				autocomplete="new-password"
			/>
			<button
				type="button"
				class="toggle-password"
				on:click={() => (showPassword = !showPassword)}
				aria-label="Mostrar u ocultar contraseña"
			>
				{showPassword ? 'Ocultar' : 'Mostrar'}
			</button>
		</div>
		{#if passwordStrength}
			<p class="strength-indicator">
				Fortaleza: <span class={`strength ${passwordStrength.toLowerCase()}`}>{passwordStrength}</span>
			</p>
		{/if}
		{#if formErrors.password}
			<p class="error-message">⚠️ {formErrors.password}</p>
		{/if}
	</div>

	<button type="submit" class="submit-btn">Registrar</button>
</form>

<style>
	/* estilos iguales al código anterior */

	.form-container {
		max-width: 480px;
		margin: 2rem auto;
		padding: 2rem 2.5rem;
		background: #fff;
		border-radius: 10px;
		box-shadow: 0 6px 18px rgba(0, 0, 0, 0.07);
		font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
		color: #333;
		transition: background-color 0.4s ease, color 0.4s ease, box-shadow 0.4s ease;
	}

	.form-container.dark-mode {
		background: #121212;
		color: #e1e1e1;
		box-shadow: 0 6px 18px rgba(255 255 255 / 0.1);
	}

	.form-title {
		text-align: center;
		margin-bottom: 1.5rem;
		color: #20c997;
		font-weight: 700;
		font-size: 1.8rem;
		transition: color 0.4s ease;
	}

	.form-container.dark-mode .form-title {
		color: #0dc9a0;
	}

	.form-group {
		margin-bottom: 1.25rem;
		display: flex;
		flex-direction: column;
	}

	label {
		font-weight: 600;
		margin-bottom: 0.4rem;
		font-size: 1rem;
		color: #0056b3;
		transition: color 0.4s ease;
	}

	.form-container.dark-mode label {
		color: #7fffd4;
	}

	input[type='text'],
	input[type='tel'],
	input[type='password'],
	input[type='file'],
	input[name='nombre'],
	input[name='direccion'] {
		padding: 0.65rem 1rem;
		border: 2px solid rgba(203, 213, 225, 0.6);
		border-radius: 8px;
		font-size: 1rem;
		transition: border-color 0.4s ease, box-shadow 0.3s ease;
	}

	.form-container.dark-mode input[type='text'],
	.form-container.dark-mode input[type='tel'],
	.form-container.dark-mode input[type='password'],
	.form-container.dark-mode input[type='file'],
	.form-container.dark-mode input[name='nombre'],
	.form-container.dark-mode input[name='direccion'] {
		background: #1e1e1e;
		border-color: rgba(100, 149, 237, 0.3);
		color: #ddd;
	}

	input:focus {
		border-color: #20c997;
		box-shadow: 0 0 8px rgba(32, 201, 151, 0.6);
		outline: none;
		transition: border-color 0.4s ease, box-shadow 0.4s ease;
	}

	button.toggle-password {
		background: transparent;
		border: none;
		color: #20c997;
		font-weight: 600;
		cursor: pointer;
		padding: 0 10px;
		user-select: none;
		transition: color 0.3s ease;
	}

	button.toggle-password:hover {
		color: #13875f;
	}

	.password-wrapper {
		display: flex;
		align-items: center;
	}

	.strength-indicator {
		font-size: 0.9rem;
		margin-top: 0.3rem;
	}

	.strength {
		font-weight: 700;
		padding: 0 0.5rem;
		border-radius: 5px;
		color: white;
	}

	.strength.débil {
		background: #dc3545;
	}

	.strength.media {
		background: #ffc107;
		color: #222;
	}

	.strength.fuerte {
		background: #20c997;
	}

	button.submit-btn {
		width: 100%;
		background-color: #0056b3;
		color: white;
		font-size: 1.1rem;
		font-weight: 700;
		padding: 0.75rem 0;
		border: none;
		border-radius: 10px;
		box-shadow: 0 6px 12px rgba(0, 86, 179, 0.5);
		cursor: pointer;
		transition: background-color 0.4s ease, box-shadow 0.3s ease;
	}

	button.submit-btn:hover {
		background-color: #003d80;
		box-shadow: 0 8px 16px rgba(0, 61, 128, 0.7);
	}

	.error-message {
		color: #dc3545;
		font-size: 0.9rem;
		margin-top: 0.25rem;
	}

	.success-message {
		color: #20c997;
		font-weight: 700;
		text-align: center;
		font-size: 1.2rem;
		margin-bottom: 1rem;
	}

	.mode-toggle-container {
		text-align: center;
		margin: 1rem 0;
	}

	.mode-toggle-btn {
		background-color: #20c997;
		color: white;
		border: none;
		padding: 0.5rem 1rem;
		font-weight: 600;
		font-size: 1rem;
		border-radius: 25px;
		cursor: pointer;
		transition: background-color 0.3s ease;
		user-select: none;
	}

	.mode-toggle-btn:hover {
		background-color: #13875f;
	}

	.form-container.dark-mode .mode-toggle-btn {
		background-color: #0dc9a0;
		color: #121212;
	}

	.form-container.dark-mode .mode-toggle-btn:hover {
		background-color: #0a6d68;
	}
</style>
