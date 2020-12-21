# Ansible-SNOW
Demos for Ansible Use in connecting with ServiceNow

# Templates

ServiceNowCatalog.yml
> Used to update a request from the self service catalog to provide closed loop automation

ServiceNowticket.yml
> Create an incident ticket in servicenow. User must supply {{ incident_description }}, {{ sn_urgency }}, and {{ sn_impact }}. Ansible uses facts stored in the inventory for ticket addition (need custom fields in ServiceNow)

ServiceNowticket Updating.yml
> Update an incident ticket in Servicenow. User must supply {{ ticket_number }} and {{ state }}. {{ state }} is based on state information contained in service now and should be a number