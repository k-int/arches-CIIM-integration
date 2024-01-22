# Implementing Arches views for CIIM 

## Steps
1. Copy the file "ciimchanges.py" into the Arches project views directory (it is entirely possible there isn't a views folder in the project so in that case simply make one).       
This contains the core code for exporting all ConceptSchemes to XML and displaying resource changes from a given date.

2. Navigate to the project "urls.py" and add the following code, first a new import to the top of the file:
```
from .views.ciimchanges import ChangesView, ConceptsExportView
```
Secondly, add the following to urlpatterns:
```
url(r"^resource/changes", ChangesView.as_view(), name="ChangesView"),
url(r"^concept/export", ConceptsExportView.as_view(), name="ConceptsExportView"),
```
3. In the core Arches models/models.py the LatestResourceEdit database table needs to be created by adding the following:
```
class LatestResourceEdit(models.Model):
    editlogid = models.UUIDField(primary_key=True, default=uuid.uuid1)
    username = models.TextField(blank=True, null=True)
    resourcedisplayname = models.TextField(blank=True, null=True)
    resourceinstanceid = models.TextField(blank=True, null=True)
    edittype = models.TextField(blank=True, null=True)
    timestamp = models.DateTimeField(blank=True, null=True)

    class Meta:
        managed = True
        db_table = "latest_resource_edit"
```
4. In models/resource.py, at the top of the file replace the previous EditLog import with the following line:
```
from arches.app.models.models import EditLog, LatestResourceEdit
```
Then, underneath edit.save() in the function save_edit add the following:
```
        if LatestResourceEdit.objects.filter(resourceinstanceid=self.resourceinstanceid).exists():
            LatestResourceEdit.objects.get(resourceinstanceid=self.resourceinstanceid).delete()
        latest_edit = LatestResourceEdit()
        latest_edit.resourceinstanceid = self.resourceinstanceid
        latest_edit.timestamp = timestamp
        latest_edit.edittype = edit_type
        latest_edit.save()
```

5. In models/tile.py, at the top of the file replace the previous EditLog import with the following line:
```
from arches.app.models.resource import EditLog, LatestResourceEdit
```
Then once again, underneat edit.save() in the function save_edit add the following:
```
        if LatestResourceEdit.objects.filter(resourceinstanceid=self.resourceinstance.resourceinstanceid).exists():
            LatestResourceEdit.objects.get(resourceinstanceid=self.resourceinstance.resourceinstanceid).delete()
        latest_edit = LatestResourceEdit()
        latest_edit.resourceinstanceid = self.resourceinstance.resourceinstanceid
        latest_edit.timestamp = timestamp
        latest_edit.edittype = edit_type
        latest_edit.save()
```
6. Migrate the model changes by running the following commands:
```
python manage.py makemigrations
python manage.py migrate
```