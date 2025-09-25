# Workflow draft for Exolynk MVP

- name: wf_sales_to_picking
  trigger: sales.to_picking
  guard:
    - role == "Sales"
    - state == "NEW"
  conditions:
    - if amount_total <= 10000 then state = "PICKING"
    - else state = "APPROVAL"
  actions:
    - status = "EXECUTION"
    - notify: svc_notify_logistics_picking when state == "PICKING"
    - notify: svc_notify_manager_approval when state == "APPROVAL"

- name: wf_manager_allow_picking
  trigger: manager.allow_picking
  guard:
    - role == "Manager"
    - state == "APPROVAL"
  actions:
    - state = "PICKING"
    - notify: svc_notify_logistics_picking

- name: wf_logistics_ready
  trigger: logistics.ready
  guard:
    - role == "Logistics"
    - state == "PICKING"
  actions:
    - validate ship_date not null
    - state = "READY"
    - shipment_state = "READY"
    - notify: svc_notify_manager_payment

- name: wf_manager_confirm_payment
  trigger: manager.confirm_payment
  guard:
    - role == "Manager"
    - state == "READY"
  actions:
    - payment_state = "YES"
    - call: svc_close_success

- name: wf_sales_close_fail
  trigger: sales.close_fail
  guard:
    - role == "Sales"
    - state != "CLOSED"
  actions:
    - ask: closed_reason
    - state = "CLOSED"
    - status = "CLOSED"
    - closed_success = false
    - notify: svc_close_success (failure template)
