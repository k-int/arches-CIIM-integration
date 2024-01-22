# Implementing Arches views for CIIM 

## Steps
1. Copy the file "ciimchanges.py" into the Arches project views directory (it is entirely possible there isn't a views folder in the project so in that case simply make one).       
This contains the core code for the following two APIs:
- Exporting all ConceptSchemes to XML.
- Displaying resource changes from a given date.

2. Navigate to the project "urls.py" to add the following code, but first a new import to the top of the file:
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
Then once again, underneath edit.save() in the function save_edit add the following:
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

7. To initiate the use of the latest_resource_edits table, a management command must be run. This command can be found in this repo under the name `populate_latest_resource_edit_table.py`. Run the command, with the virtual environment activated, using 
```
python manage.py populate_latest_resource_edit_table
```


For more documentation, see the shared Google Drive folder: https://drive.google.com/drive/u/0/folders/1LV-7YT5DHExh5NaodRncn5ODjR2Wun-h