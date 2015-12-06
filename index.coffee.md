# Google Drive text editor

## Imports

	querystring = require 'querystring'


## Document ready

	document.addEventListener 'DOMContentLoaded', ->
		document.getElementById 'save_button'
		.addEventListener 'click', (event)->
			event.preventDefault()

			console.log 'Save button clicked.'

			get_file_id_from_url 'id'

			.then (file_id)->
				get_editor_text_content()

				.then (content)->
					update_file_in_drive file_id, content

				.then (file)->
					console.log 'File updated!'
					console.log file

			.catch (err)->
				console.error err

			false


## Get editor text content

	get_editor_text_content = ->
		Promise.resolve document.getElementById('file_content').value


## Get file ID from URL

There may be multiple ID values, separated by commas.

	get_file_id_from_url = (key)->
		query = querystring.parse document.location.search.slice 1

		if query[key]
			Promise.resolve query[key]
		else
			Promise.reject 'file_id not found in URL.'


## Google client code loaded

This is called when the Google client API script has finished loading.
`google_client_onload` is included in the `src` attribute of the `script` tag in "index.html", given as a callback.

	window.google_client_onload = ->
		console.log 'Checking if already authorised.'

		auth =
			client_id: '929534383010-hui1c2g21lphi0o9tg4rblmlqb1v4fo8.apps.googleusercontent.com'
			immediate: true
			scope: 'https://www.googleapis.com/auth/drive'
			#scope: 'https://www.googleapis.com/auth/drive.file'
			#scope: 'https://www.googleapis.com/auth/drive.metadata.readonly'

		auth_good = ->
			console.log 'Loading Google Drive API.'

			gapi.client.load 'drive', 'v2', ->
				console.log 'Loading file from URL.'

				get_file_id_from_url 'id'
				.then load_file_metadata
				.then load_file_content
				.then update_editor_with_content
				.catch (err)->
					console.error err

		gapi.auth.authorize auth
		, (auth_result)->
			if auth_result and not auth_result.error
				console.log 'Already authorised!'

				auth_good()

			else
				console.log 'Not already authorised. Doing that now.'

				auth.immediate = false

				gapi.auth.authorize auth
				, (auth_result)->
					if not auth_result
						console.error 'Invalid response from `gapi.auth.authorize()`!'

					else if auth_result.error
						console.error auth_result.error

					else
						console.log 'Authorised!'

						auth_good()


## Load file content

	load_file_content = (file)->
		new Promise (resolve, reject)->
			access_token = gapi.auth.getToken().access_token

			xhr = new XMLHttpRequest()

			xhr.onerror = ->
				reject 'Failed to load file content!'

			xhr.onload = ->
				resolve xhr.responseText

			xhr.open 'GET', file.downloadUrl

			xhr.setRequestHeader 'Authorization', "Bearer #{access_token}"

			xhr.send()


## Load file metadata

Callback receives an object with the following fields:

- description

- downloadUrl

- mimeType

- title

	load_file_metadata = (file_id)->
		console.log "Loading metadata for file ID: #{file_id}"

		new Promise (resolve, reject)->
			request = gapi.client.drive.files.get fileId: file_id
			request.execute (file)->
				if not file
					reject 'No file metadata returned.'

				else if file.error
					#if file.error.code == 401
						#reject 'Access token may have expired.'

					reject file.error

				else
					resolve file


## Update editor with content

	update_editor_with_content = (content)->
		document.getElementById 'file_content'
		.value = content


## Update file

	update_file_in_drive = (file_id, content)->
		new Promise (resolve, reject)->
			console.log 'Sending Drive update file request.'

			gapi.client.request
				body: content
				#headers:
					#'Content-Length': content.length
					#'Content-Type': 'application/json; charset=UTF-8'
				method: 'PUT'
				params:
					#alt: 'json'
					uploadType: 'media'
				path: "/upload/drive/v2/files/#{file_id}"

			.execute (file)->
				if not file
					reject 'No file upload details returned.'

				else if file.error
					#if file.error.code == 401
						#reject 'Access token may have expired.'

					reject file.error

				else
					resolve file


## Links

### Auth

- [Authorizing Requests for Web Applications @ Google Developers](https://developers.google.com/drive/web/auth/web-client)

- [Enable the Drive Platform @ Google Developers](https://developers.google.com/drive/web/enable-sdk)

- [JavaScript Quickstart @ Google Developers](https://developers.google.com/drive/web/quickstart/js)


### Download and upload overviews

- [Download Files @ Google Developers](https://developers.google.com/drive/web/manage-downloads)

- [Upload Files @ Google Developers](https://developers.google.com/drive/web/manage-uploads)


### Reference

- [Files: get @ Google Developers](https://developers.google.com/drive/v2/reference/files/get)

- [Files: update @ Google Developers](https://developers.google.com/drive/v2/reference/files/update)
