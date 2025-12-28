.PHONY: deploy
deploy:
	@helmfile -e $(ENV) init
	@helmfile -e $(ENV) diff
	@helmfile -e $(ENV) apply
	@helmfile -e $(ENV) status
	@kubectl apply -f manifests/namespace.yaml
	@kubectl apply -f manifests/app-of-apps-$(ENV).yaml
