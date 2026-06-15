Dataset
- ID -> Pkey Integer Auto Increment
- Dataset_id -> UUID time encoded Non Null
- Name -> Varchar Non Null
- Description -> Text preferably non null
- Created_at -> Datetime
- Updated_at -> Datetime
- Archived_at -> Datetime
- Active -> Boolean

Metadata
- dataset -> FKEY to Dataset
- metadata -> JSONB
- Created_at -> Datetime
- Updated_at -> Datetime
- Archived_at -> Datetime
- Active -> Boolean

Columns
- dataset -> Fkey to Dataset
- columns -> JSONB
	- {[
		name of the column
		type of the column
	],
	[
		name of the column
		type of the column
	],
	}
- Created_at -> Datetime
- Updated_at -> Datetime
- Archived_at -> Datetime
- Active -> Boolean

Dataset Relations (Through table to understand which dataset is related to which table and how)'
- master_dataset_id -> Fkey to Dataset Table (Represent the master dataset)
- variant_dataset_id -> Fkey to Dataset Table (Represents the variant dataset)
- Changes -> JSONB (Representing the diff between the master columns and variant columns)

Note all these need to be git like in the 