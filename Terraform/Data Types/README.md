# 🔹 1. string

Text values (characters)

variable "name" {
  type = string
  default = "Sahil"
}

👉 Used for names, IDs, tags, etc.

# 🔹 2. number

Numeric values (integer or decimal)

variable "age" {
  type = number
  default = 25
}

👉 Used for counts, sizes, ports, etc.

# 🔹 3. bool (boolean)

True or false values

variable "is_active" {
  type = bool
  default = true
}

👉 Used for enabling/disabling features

# 🔹 4. list

Ordered collection of same type values

variable "servers" {
  type = list(string)
  default = ["web1", "web2", "web3"]
}

👉 Order matters (index-based access)

servers[0]  # web1
# 🔹 5. map

Key-value pairs (same type values)

variable "user" {
  type = map(string)
  default = {
    name = "Sahil"
    city = "Pune"
  }
}

👉 Access using key

user["name"]  # Sahil
# 🔹 6. map(number)

Map where all values are numbers

variable "ports" {
  type = map(number)
  default = {
    http  = 80
    https = 443
  }
}

👉 Useful for configurations like ports, limits

# 🔹 7. set

Unordered collection of unique values

variable "unique_ips" {
  type = set(string)
  default = ["10.0.0.1", "10.0.0.2", "10.0.0.1"]
}

👉 Output will remove duplicates:

["10.0.0.1", "10.0.0.2"]

⚠️ No indexing (cannot use [0])
# 🔹 8. null

Represents absence of a value

variable "description" {
  type = string
  default = null
}

👉 Useful when:

You want to skip a value
Let Terraform use default behavior

Example:

resource "aws_instance" "example" {
  instance_type = "t2.micro"
  key_name      = null   # will not set key
}
