

## Content-type
raw->json
	Content-Type: application/json
	Req-body
	{
		"a": "1"
	}

raw->text
	Content-Type: application/text
	Req-body
	{
		"a": "1"
	}

raw->xml
	Content-Type: application/xml
	Req-body
	{
		"a": "1"
	}


form
	Content-Type: multipart/form-data; boundary=--------------------------058363263533734731568425
	Req-body
	a: "1"


x-www-form-urlencoded
	Content-Type: application/x-www-form-urlencoded
	Req-body
	a: "1"