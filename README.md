# Wallet


<h2>Device Table</h2>

| Attribute               	| Type   	| Description                                                	|
|-------------------------	|--------	|------------------------------------------------------------	|
| id                      	| int    	| Primary Key                                                	|
| deviceLibraryIdentifier 	| String 	| Uniqure ID to identify and authenticate a device           	|
| pushToken               	| String 	| Push token used to send update notifications to the device 	|


<h2>Pass Table</h2>

| Attribute          | Type     | Description                                                       |
| ------------------ | -------- | ----------------------------------------------------------------- |
| id                 | `int`    | Primary key                                                       |
| passTypeIdentifier | `string` | Your pass type identifier                                         |
| serialNumber       | `string` | Unique identifier for the pass                                    |
| lastUpdateTag      | `date`   | Timestamp of the last pass update                                 |

<h2>Registration Table</h2>
<p>Manages the many-to-many relationship between devices and passes.</p>
| Attribute                | Type     | Description                               |
| ------------------------ | -------- | ----------------------------------------- |
| id                       | `int`    | Primary key                               |
| deviceId                 | `int`    | Foreign key referencing the Device table  |
| passId                   | `int`    | Foreign key referencing the Pass table    |


<h2>Device Registration Process</h2>
When a device attempts to register, your server should:

1. Authenticate the request using the provided `authenticationToken`
2. Find or create the device record using the `deviceLibraryIdentifier` and `pushToken`
3. Find or create the pass record using the `passTypeIdentifier` and `serialNumber`
4. Find or create the registration relationship between the device and pass
5. Return appropriate status codes:
- `200`: Pass already registered
- `201`: Registration successful
- `401`: Unauthorized request

Beispielcode für eine Registrierung:

`export const POST = async (request: NextRequest, { params }: { params: PathParams }): Promise<Response> => {
  const { deviceLibraryIdentifier, passTypeIdentifier, serialNumber } = params;
  const { pushToken } = await request.json();
  const authHeader = request.headers.get("Authorization");

  if (authHeader !== `ApplePass ${process.env.AUTH_TOKEN}`) {
    return NextResponse.json({ message: "Unauthorized" }, { status: 401 });
  }

  try {
    // Find or create device
    const device = await Devices.findOrCreate({
      deviceLibraryIdentifier,
      pushToken
    });

    // Find or create pass
    const pass = await Passes.findOrCreate({
      passTypeIdentifier,
      serialNumber
    });

    // Create registration if it doesn't exist
    const registration = await Registrations.findOrCreate({
      deviceId: device.id,
      passId: pass.id
    });

    return NextResponse.json(
      { message: registration.isNew ? "Registration created" : "Registration exists" },
      { status: registration.isNew ? 201 : 200 }
    );
  } catch (error) {
    return NextResponse.json(
      { message: "Internal Server Error" },
      { status: 500 }
    );
  }
};`

