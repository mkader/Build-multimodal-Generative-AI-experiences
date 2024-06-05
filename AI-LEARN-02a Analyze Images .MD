https://microsoftlearning.github.io/mslearn-ai-vision/Instructions/Exercises/01-analyze-images.html
AI-LEARN-02a Analyze Images (Learn)

	Analyze Images with Azure AI Vision
		Azure AI Vision is an AI capability that enables software systems to interpret visual input by analyzing images.
		In Azure, the Vision Azure AI service provides pre-built models for common computer vision tasks, including analysis of images to suggest captions and tags, detection of common objects and people.
		Use the Azure AI Vision service to remove the background or create a foreground matting of images.

	Clone the repository for this course -> VSC ->  palette (SHIFT+CTRL+P) and run a Git: Clone command to clone the https://github.com/MicrosoftLearning/mslearn-ai-vision

	Provision an Azure AI Services resource
		Azure portal -> Azure AI services -> Region: East US, West US* -> Pricing tier: Standard S0
		*Azure AI Vision 4.0 features are currently only available in these regions.

	Prepare to use the Azure AI Vision SDK (Labfiles/01-analyze-images)
		C# 	: dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
		Python	: pip install azure-ai-vision-imageanalysis==1.0.0b1

		C#: appsettings.json
		Python: .env

		C#: Program.cs
		Python: image-analysis.py

		C#
		// Import namespaces
		using Azure.AI.Vision.ImageAnalysis;

		Python
		# import namespaces
		from azure.ai.vision.imageanalysis import ImageAnalysisClient
		from azure.ai.vision.imageanalysis.models import VisualFeatures
		from azure.core.credentials import AzureKeyCredential

	View the images you will analyze ( use the Azure AI Vision service to analyze multiple images).

	Analyze an image to suggest a caption
		C#
		// Authenticate Azure AI Vision client
		ImageAnalysisClient client = new ImageAnalysisClient(
		    new Uri(aiSvcEndpoint),
		    new AzureKeyCredential(aiSvcKey));

		Python
		# Authenticate Azure AI Vision client
		cv_client = ImageAnalysisClient(
		    endpoint=ai_endpoint,
		    credential=AzureKeyCredential(ai_key)
		)

		C#
		// Get result with specified features to be retrieved
		ImageAnalysisResult result = client.Analyze(
		    BinaryData.FromStream(stream),
		    VisualFeatures.Caption |
		    VisualFeatures.DenseCaptions |
		    VisualFeatures.Objects |
		    VisualFeatures.Tags |
		    VisualFeatures.People);

		Python
		# Get result with specified features to be retrieved
		result = cv_client.analyze(
		    image_data=image_data,
		    visual_features=[
			VisualFeatures.CAPTION,
			VisualFeatures.DENSE_CAPTIONS,
			VisualFeatures.TAGS,
			VisualFeatures.OBJECTS,
			VisualFeatures.PEOPLE],
		)

		C#
		// Display analysis results
		// Get image captions
		if (result.Caption.Text != null)
		{
		    Console.WriteLine(" Caption:");
		    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
		}

		// Get image dense captions
		Console.WriteLine(" Dense Captions:");
		foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
		{
		    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
		}

		// Get image tags
		// Get objects in the image
		// Get people in the image

		Python

		Python
		# Display analysis results
		# Get image captions
		if result.caption is not None:
		    print("\nCaption:")
		    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

		# Get image dense captions
		if result.dense_captions is not None:
		    print("\nDense Captions:")
		    for caption in result.dense_captions.list:
			print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

		# Get image tags
		# Get objects in the image
		# Get people in the image

		Save & Run "dotnet run images/street.jpg" "python image-analysis.py images/street.jpg". Run the program building.jpg, person.jpg
		
		Observe the output, which should include a suggested caption for the street.jpg image.
		 Caption:
		   "a man walking a dog on a leash on a street", Confidence 0.82

		 Dense Captions:
		   Caption: 'a man walking a dog on a leash on a street', Confidence: 0.82
		   Caption: 'a man walking on a street', Confidence: 0.69
		   Caption: 'a yellow car on the street', Confidence: 0.78
		   Caption: 'a black dog walking on the street', Confidence: 0.75
		   Caption: 'a blurry image of a blue car', Confidence: 0.82
   		   Caption: 'a yellow taxi cab on the street', Confidence: 0.72

	Get suggested tags for an image
		C#
		// Get image tags
		if (result.Tags.Values.Count > 0)
		{
		    Console.WriteLine($"\n Tags:");
		    foreach (DetectedTag tag in result.Tags.Values)
		    {
			Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
		    }
		}

		Python
		# Get image tags
		if result.tags is not None:
		    print("\nTags:")
		    for tag in result.tags.list:
			print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
			
			
		Tags:
		   'outdoor', Confidence: 1.00
		   'land vehicle', Confidence: 0.99
		   'vehicle', Confidence: 0.99
		   'building', Confidence: 0.99
		   'road', Confidence: 0.96
		   'wheel', Confidence: 0.95
		   'street', Confidence: 0.95
		   'person', Confidence: 0.93
		   'clothing', Confidence: 0.91
		   'taxi', Confidence: 0.91
		   'car', Confidence: 0.84
		   'dog', Confidence: 0.83
		   'yellow', Confidence: 0.77
		   'walking', Confidence: 0.74
		   'city', Confidence: 0.65
		   'woman', Confidence: 0.58	

	Detect and locate objects in an image
		C#
		// Get objects in the image
		if (result.Objects.Values.Count > 0)
		{
		    Console.WriteLine(" Objects:");

		    // Prepare image for drawing
		    stream.Close();
		    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
		    Graphics graphics = Graphics.FromImage(image);
		    Pen pen = new Pen(Color.Cyan, 3);
		    Font font = new Font("Arial", 16);
		    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

		    foreach (DetectedObject detectedObject in result.Objects.Values)
		    {
			Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

			// Draw object bounding box
			var r = detectedObject.BoundingBox;
			Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
			graphics.DrawRectangle(pen, rect);
			graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
		    }

		    // Save annotated image
		    String output_file = "objects.jpg";
		    image.Save(output_file);
		    Console.WriteLine("  Results saved in " + output_file + "\n");
		}

		Python
		# Get objects in the image
		if result.objects is not None:
		    print("\nObjects in image:")

		    # Prepare image for drawing
		    image = Image.open(image_filename)
		    fig = plt.figure(figsize=(image.width/100, image.height/100))
		    plt.axis('off')
		    draw = ImageDraw.Draw(image)
		    color = 'cyan'

		    for detected_object in result.objects.list:
			# Print object name
			print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))

			# Draw object bounding box
			r = detected_object.bounding_box
			bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
			draw.rectangle(bounding_box, outline=color, width=3)
			plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

		    # Save annotated image
		    plt.imshow(image)
		    plt.tight_layout(pad=0)
		    outputfile = 'objects.jpg'
		    fig.savefig(outputfile)
		    print('  Results saved in', outputfile)
		    
		 Objects:
		   "car"
		   "taxi"
		   "person"
		   "dog"		    
		    
	Detect and locate people in an image
		C#
		// Get people in the image
		if (result.People.Values.Count > 0)
		{
		    Console.WriteLine($" People:");

		    // Prepare image for drawing
		    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
		    Graphics graphics = Graphics.FromImage(image);
		    Pen pen = new Pen(Color.Cyan, 3);
		    Font font = new Font("Arial", 16);
		    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

		    foreach (DetectedPerson person in result.People.Values)
		    {
			// Draw object bounding box
			var r = person.BoundingBox;
			Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
			graphics.DrawRectangle(pen, rect);

			// Return the confidence of the person detected
			//Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
		    }

		    // Save annotated image
		    String output_file = "persons.jpg";
		    image.Save(output_file);
		    Console.WriteLine("  Results saved in " + output_file + "\n");
		}

		Python
		# Get people in the image
		if result.people is not None:
		    print("\nPeople in image:")

		    # Prepare image for drawing
		    image = Image.open(image_filename)
		    fig = plt.figure(figsize=(image.width/100, image.height/100))
		    plt.axis('off')
		    draw = ImageDraw.Draw(image)
		    color = 'cyan'

		    for detected_people in result.people.list:
			# Draw object bounding box
			r = detected_people.bounding_box
			bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
			draw.rectangle(bounding_box, outline=color, width=3)

			# Return the confidence of the person detected
			#print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))

		    # Save annotated image
		    plt.imshow(image)
		    plt.tight_layout(pad=0)
		    outputfile = 'people.jpg'
		    fig.savefig(outputfile)
		    print('  Results saved in', outputfile)
		    
	 People:
	   Bounding box Top: 109, Left: 241, Width: 155, Height: 399, Confidence: 0.95
	   Bounding box Top: 264, Left: 396, Width: 23, Height: 58, Confidence: 0.25
	   Bounding box Top: 262, Left: 699, Width: 20, Height: 33, Confidence: 0.22
	   Bounding box Top: 230, Left: 138, Width: 28, Height: 31, Confidence: 0.07
	   Bounding box Top: 188, Left: 129, Width: 15, Height: 26, Confidence: 0.01
	   Bounding box Top: 188, Left: 146, Width: 16, Height: 25, Confidence: 0.01
	   Bounding box Top: 263, Left: 405, Width: 15, Height: 28, Confidence: 0.01
	   Bounding box Top: 187, Left: 164, Width: 17, Height: 25, Confidence: 0.01
	   Bounding box Top: 187, Left: 176, Width: 13, Height: 25, Confidence: 0.00
	   Bounding box Top: 186, Left: 139, Width: 42, Height: 29, Confidence: 0.00
	   Bounding box Top: 262, Left: 510, Width: 17, Height: 10, Confidence: 0.00
	   Bounding box Top: 247, Left: 247, Width: 141, Height: 121, Confidence: 0.00
	  Results saved in persons.jpg	    
		    

	Note: In the preceding tasks, you used a single method to analyze the image, and then incrementally added code to parse and display the results. 
	The SDK also provides individual methods for suggesting captions, identifying tags, detecting objects, and so on - meaning that use the most appropriate method to return only the information you need, reducing the size of the data payload that needs to be returned. 
	
	Remove the background or generate a foreground matte of an image
		C#
		// Remove the background from the image or generate a foreground matte
		Console.WriteLine($" Background removal:");
		// Define the API version and mode
		string apiVersion = "2023-02-01-preview";
		string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

		string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

		// Make the REST call
		using (var client = new HttpClient())
		{
		    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
		    client.BaseAddress = new Uri(endpoint);
		    client.DefaultRequestHeaders.Accept.Add(contentType);
		    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

		    var data = new
		    {
			url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
		    };

		    var jsonData = JsonSerializer.Serialize(data);
		    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
		    var response = await client.PostAsync(url, contentData);

		    if (response.IsSuccessStatusCode) {
			File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
			Console.WriteLine("  Results saved in background.png\n");
		    }
		    else
		    {
			Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
		    }
		}
	
		Python
		# Remove the background from the image or generate a foreground matte
		print('\nRemoving background from image...')

		url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

		headers= {
		    "Ocp-Apim-Subscription-Key": key,
		    "Content-Type": "application/json"
		}

		image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)

		body = {
		    "url": image_url,
		}

		response = requests.post(url, headers=headers, json=body)

		image=response.content
		with open("backgroundForeground.png", "wb") as file:
		    file.write(image)
		print('  Results saved in backgroundForeground.png \n')

