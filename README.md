using System;
using System.Collections.Generic;
using System.IO;

namespace HotelManagement
{
    class Room
    {
        public int RoomNumber { get; set; }
        public bool IsAllocated { get; set; } = false;
        public string GuestName { get; set; } = "";
    }

    class Program
    {
        static List<Room> rooms = new List<Room>();
        static string dataFile = "lhms_studentid.txt";
        static string backupFile = "lhms_studentid_backup.txt";

        static void Main(string[] args)
        {
            int choice;
            do
            {
                Console.WriteLine("\n*** LANGHAM HOTEL MANAGEMENT SYSTEM ***");
                Console.WriteLine("1. Add Rooms\n2. Display Rooms\n3. Allocate Rooms\n4. De-Allocate Rooms\n5. Display Room Allocation Details\n6. Billing\n7. Save the Room Allocations To a File\n8. Show the Room Allocations From a File\n9. Exit");
                Console.Write("Enter your choice: ");
                try
                {
                    choice = int.Parse(Console.ReadLine());
                    switch (choice)
                    {
                        case 1: AddRooms(); break;
                        case 2: DisplayRooms(); break;
                        case 3: AllocateRoom(); break;
                        case 4: DeallocateRoom(); break;
                        case 5: DisplayAllocation(); break;
                        case 6: Console.WriteLine("Billing Feature is Under Construction and will be added soon...!!!"); break;
                        case 7: SaveToFile(); break;
                        case 8: ShowFromFile(); break;
                        case 9: Console.WriteLine("Exiting..."); break;
                        default: Console.WriteLine("Invalid Choice"); break;
                    }
                }
                catch (FormatException)
                {
                    Console.WriteLine("Invalid input format. Please enter a number.");
                    choice = 0;
                }
            } while (choice != 9);
        }

        static void AddRooms()
        {
            try
            {
                Console.Write("Enter number of rooms to add: ");
                int count = int.Parse(Console.ReadLine());
                for (int i = 0; i < count; i++)
                {
                    Console.Write($"Enter room number for room {i + 1}: ");
                    int rno = int.Parse(Console.ReadLine());
                    rooms.Add(new Room { RoomNumber = rno });
                }
            }
            catch (FormatException)
            {
                Console.WriteLine("Room number must be numeric.");
            }
        }

        static void DisplayRooms()
        {
            if (rooms.Count == 0)
            {
                Console.WriteLine("No rooms added yet.");
                return;
            }
            foreach (var room in rooms)
            {
                Console.WriteLine($"Room {room.RoomNumber} - {(room.IsAllocated ? "Allocated" : "Available")}");
            }
        }

        static void AllocateRoom()
        {
            try
            {
                Console.Write("Enter Room Number to allocate: ");
                int rno = int.Parse(Console.ReadLine());
                Room room = rooms.Find(r => r.RoomNumber == rno);
                if (room != null && !room.IsAllocated)
                {
                    Console.Write("Enter Guest Name: ");
                    room.GuestName = Console.ReadLine();
                    room.IsAllocated = true;
                    Console.WriteLine("Room allocated successfully.");
                }
                else
                {
                    throw new InvalidOperationException("Room not found or already allocated.");
                }
            }
            catch (FormatException)
            {
                Console.WriteLine("Please enter a numeric value for Room Number.");
            }
            catch (InvalidOperationException e)
            {
                Console.WriteLine($"Error: {e.Message}");
            }
        }

        static void DeallocateRoom()
        {
            try
            {
                Console.Write("Enter Room Number to deallocate: ");
                int rno = int.Parse(Console.ReadLine());
                Room room = rooms.Find(r => r.RoomNumber == rno);
                if (room != null && room.IsAllocated)
                {
                    room.IsAllocated = false;
                    room.GuestName = "";
                    Console.WriteLine("Room deallocated successfully.");
                }
                else
                {
                    throw new InvalidOperationException("Room not found or already available.");
                }
            }
            catch (FormatException)
            {
                Console.WriteLine("Please enter a valid number.");
            }
            catch (InvalidOperationException e)
            {
                Console.WriteLine($"Error: {e.Message}");
            }
        }

        static void DisplayAllocation()
        {
            bool anyAllocated = false;
            foreach (var room in rooms)
            {
                if (room.IsAllocated)
                {
                    Console.WriteLine($"Room {room.RoomNumber} - Guest: {room.GuestName}");
                    anyAllocated = true;
                }
            }

            if (!anyAllocated)
                Console.WriteLine("No rooms are currently allocated.");
        }

        static void SaveToFile()
        {
            try
            {
                using (StreamWriter sw = new StreamWriter(dataFile, true))
                {
                    foreach (var room in rooms)
                    {
                        if (room.IsAllocated)
                        {
                            sw.WriteLine($"[{DateTime.Now}] Room {room.RoomNumber}: {room.GuestName}");
                        }
                    }
                }
                Console.WriteLine("Room allocations saved to file successfully.");
            }
            catch (UnauthorizedAccessException)
            {
                Console.WriteLine("Access denied to write the file.");
            }
            catch (Exception e)
            {
                Console.WriteLine($"Unexpected error: {e.Message}");
            }
        }

        static void ShowFromFile()
        {
            try
            {
                if (!File.Exists(dataFile))
                    throw new FileNotFoundException();

                string[] lines = File.ReadAllLines(dataFile);

                if (lines.Length == 0)
                {
                    Console.WriteLine("No data to show in file.");
                    return;
                }

                Console.WriteLine("\nRoom Allocation Details from File:");
                foreach (var line in lines)
                    Console.WriteLine(line);

                File.AppendAllLines(backupFile, lines); // backup
                File.WriteAllText(dataFile, ""); // clear original file
                Console.WriteLine("Data backed up and cleared from original file.");
            }
            catch (FileNotFoundException)
            {
                Console.WriteLine("Data file not found.");
            }
            catch (UnauthorizedAccessException)
            {
                Console.WriteLine("No permission to access the file.");
            }
            catch (Exception e)
            {
                Console.WriteLine($"Unexpected error: {e.Message}");
            }
        }
    }
}
