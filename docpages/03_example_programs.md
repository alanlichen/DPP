# Example Programs

The best way to experiment with these example programs is to delete the content from `test.cpp` in the library repository, and replace it with program code as shown below. You can then use `cmake` and `make` to build the bot without having to mess around with installation, dependencies etc.

* \subpage firstbot "Creating Your First Bot"
* \subpage slashcommands "Using Slash Commands and Interactions"
* \subpage soundboard "Creating a Sound Board"
* \subpage oggopus "Streaming Ogg Opus file"
* \subpage stream-mp3-discord-bot "Streaming MP3 files"
* \subpage record-user "Record yourself in a VC"
* \subpage joinvc "Join or switch to the voice channel of the user issuing a command"
* \subpage spdlog "Integrating with spdlog"
* \subpage components "Using component interactions (buttons)"
* \subpage components3 "Using component interactions (select menus)"
* \subpage components2 "Using component interactions (advanced)"
* \subpage modal-dialog-interactions "Modal Dialog Interactions"
* \subpage commandhandler "Using a command handler object"
* \subpage subcommands "Using sub-commands in slash commands"
* \subpage embed-message "Sending Embeds"
* \subpage application-command-autocomplete "Slash command auto completion"
* \subpage attach-file "Attaching a file"
* \subpage caching-messages "Caching messages"
* \subpage collecting-reactions "Collecting Reactions"
* \subpage context-menu "Context Menus"


\page firstbot Creating Your First Bot

In this example we will create a C++ version of the [discord.js](https://discord.js.org/#/) example program.

The two programs can be seen side by side below:

\image html progs.png

Let's break this program down step by step:

### 1. Start with an empty C++ program

Make sure to include the header file for the D++ library with the instruction \#include `<dpp/dpp.h>`!

~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iostream>

int main()
{
    return 0;
}
~~~~~~~~~~~~~~

### 2. Create an instance of dpp::cluster

To make use of the library you must create a dpp::cluster object. This object is the main object in your program like the `Discord.Client` object in Discord.js.

You can instantiate this class as shown below. Remember to put your bot token in the code where it says `"token"`!

~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iostream>

int main()
{
    dpp::cluster bot("token");

    return 0;
}
~~~~~~~~~~~~~~~

### 3. Attach to an event

To have a bot that does something, you should attach to some events. Let's start by attaching to the `on_ready` event (dpp::cluster::on_ready) which will notify your program when the bot is connected:

~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iostream>

int main()
{
    dpp::cluster bot("token");

    bot.on_ready([&bot](const auto & event) {
    });

    return 0;
}
~~~~~~~~~~~~~~~~

### 4. Attach to another event to reveice messages

If you want to receive messages, you should also attach your program to the `on_message_create` event (dpp::cluster::on_message_create) which is the same as the Discord.js `message` event. You add this to your program after the `on_ready` event:

~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iostream>

int main()
{
    dpp::cluster bot("token");

    bot.on_ready([&bot](const auto & event) {
    });

    bot.on_message_create([&bot](const auto & event) {
    });

    return 0;
}
~~~~~~~~~~~~~~

### 5 . Add some content to the events

Attaching to an event is a good start, but to make a bot you should actually put some program code into the events. We will add some code to the `on_ready` event to output the bot's nickname (dpp::cluster::me) and some code into the `on_message_create` to look for messages that are the text `!ping` and reply with `!pong`:

~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iostream>

int main()
{
    dpp::cluster bot("token");

    bot.on_ready([&bot](const auto & event) {
        std::cout << "Logged in as " << bot.me.username << "!\n";
    });

    bot.on_message_create([&bot](const auto & event) {
        if (event.msg.content == "!ping") {
            bot.message_create(dpp::message(event.msg.channel_id, "Pong!"));
        }
    });

    bot.start(false);
    return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~

Let's break down the code in the `on_message_create` event so that we can discuss what it is doing:

~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
if (event.msg.content == "!ping") {
	bot.message_create(dpp::message(event.msg.channel_id, "Pong!"));
}
~~~~~~~~~~~~~~~~~~~~~~~

This code is simply comparing the message content `event.msg.content` (dpp::message_create_t::content) against the value in a constant string value `"!ping"`. If they match, then the `message_create` function is called.

The `message_create` function (dpp::cluster::message_create) sends a message. There are many ways to call this function to send embed messages, upload files, and more, but for this simple demonstration we will just send some message text. The `message_create` function accepts a `dpp::message` object, which we create using two parameters:

* The channel ID to send to, which we get from `event.msg.channel_id` (dpp::message_create_t::channel_id)
* The message content, which for this demonstration we just make the static text `"Pong!"`.

### 6. Add code to start the bot!

To make the bot start, we must call the cluster::start method, e.g. in our program by using `bot.start(false)`.

The parameter which we set to false indicates if the function should return once all shards are created. Passing `false` here tells the program you do not need to do anything once `bot.start` is called, so the `return` statement directly afterwards is never reached:

~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iostream>

int main()
{
    dpp::cluster bot("token");

    bot.on_ready([&bot](const auto & event) {
        std::cout << "Logged in as " << bot.me.username << "!\n";
    });

    bot.on_message_create([&bot](const auto & event) {
        if (event.msg.content == "!ping") {
            bot.message_create(dpp::message(event.msg.channel_id, "Pong!"));
        }
    });

    bot.start(false);
    return 0;
}
~~~~~~~~~~~~~~

### 7. Run your bot

Compile your bot using `g++ -std=c++17 -o test test.cpp -ldpp` (if your .cpp file is called `test.cpp`) and run it with `./test`.

**Congratulations** - you now have a working bot written using the D++ library!

\page soundboard Creating a Sound Board

This example script shows how to send a sound file to a voice channel. A few shortcuts are taken here, for more advanced techniques for connecting to a voice channel see the tutorial \ref joinvc

~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iomanip>
#include <sstream>

int main(int argc, char const *argv[])
{
	/* Load a sound file called Robot.pcm into memory.
	 * The bot expects PCM format, which are raw sound data,
	 * 2 channel stereo, 16 bit signed 48000Hz.
	 * 
	 * You can use audacity to export these from WAV or MP3 etc.
	 * 
	 * If you wanted to send a more complicated format, you could
	 * use a separate library to decode that audio to PCM. For
	 * example purposes, a raw PCM will suffice. This PCM file can
	 * be found within the bot's github repo.
	 */
	uint8_t* robot = nullptr;
	size_t robot_size = 0;
	std::ifstream input ("../testdata/Robot.pcm", std::ios::in|std::ios::binary|std::ios::ate);
	if (input.is_open()) {
		robot_size = input.tellg();
		robot = new uint8_t[robot_size];
		input.seekg (0, std::ios::beg);
		input.read ((char*)robot, robot_size);
		input.close();
	}

	/* Setup the bot */
	dpp::cluster bot("token");

	/* Use the on_message_create event to look for commands */
	bot.on_message_create([&bot, robot, robot_size](const dpp::message_create_t & event) {

		std::stringstream ss(event.msg.content);
		std::string command;

		ss >> command;

		/* Tell the bot to join the discord voice channel the user is on. Syntax: .join */
		if (command == ".join") {
			dpp::guild * g = dpp::find_guild(event.msg.guild_id);
			if (!g->connect_member_voice(event.msg.author.id)) {
				bot.message_create(dpp::message(channel_id, "You don't seem to be on a voice channel! :("));
			}
		}

		/* Tell the bot to play the sound file 'Robot.pcm'. Syntax: .robot */
		if (command == ".robot") {
			dpp::voiceconn* v = event.from->get_voice(event.msg.guild_id);
			if (v && v->voiceclient && v->voiceclient->is_ready()) {
				v->voiceclient->send_audio_raw((uint16_t*)robot, robot_size);
			}
		}
	});

	/* Start bot */
	bot.start(false);
	return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~

\page oggopus Streaming Ogg Opus file

This example shows how to stream an Ogg Opus file to a voice channel. This example requires some additional dependencies, namely `libogg` and `opusfile`.

~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iomanip>
#include <sstream>

#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <ogg/ogg.h>
#include <opus/opusfile.h>

int main(int argc, char const *argv[])
{
	/* Load an ogg opus file into memory.
	 * The bot expects opus packets to be 2 channel stereo, 48000Hz.
	 * 
	 * You may use ffmpeg to encode songs to ogg opus:
	 * ffmpeg -i /path/to/song -c:a libopus -ar 48000 -ac 2 -vn -b:a 96K /path/to/opus.ogg 
	 */
	dpp::cluster bot("token");

	/* Use the on_message_create event to look for commands */
	bot.on_message_create([&bot](const dpp::message_create_t & event) {
		std::stringstream ss(event.msg.content);
		std::string command;
		ss >> command;

		/* Tell the bot to join the discord voice channel the user is on. Syntax: .join */
		if (command == ".join") {
			dpp::guild * g = dpp::find_guild(event.msg.guild_id);
			if (!g->connect_member_voice(event.msg.author.id)) {
				bot.message_create(dpp::message(event.msg.channel_id, "You don't seem to be on a voice channel! :("));
			}
		}

		/* Tell the bot to play the sound file */
		if (command == ".play") {
			dpp::voiceconn* v = event.from->get_voice(event.msg.guild_id);
			if (v && v->voiceclient && v->voiceclient->is_ready()) {
				ogg_sync_state oy; 
				ogg_stream_state os;
				ogg_page og;
				ogg_packet op;
				OpusHead header;
				char *buffer;

				FILE *fd;

				fd = fopen("/path/to/opus.ogg", "rb");

				fseek(fd, 0L, SEEK_END);
				size_t sz = ftell(fd);
				rewind(fd);

				ogg_sync_init(&oy);

				int eos = 0;
				int i;

				buffer = ogg_sync_buffer(&oy, sz);
				fread(buffer, 1, sz, fd);

				ogg_sync_wrote(&oy, sz);

				/**
				 * We must first verify that the stream is indeed ogg opus
				 * by reading the header and parsing it
				 */
				if (ogg_sync_pageout(&oy, &og) != 1)
				{
					fprintf(stderr,"Does not appear to be ogg stream.\n");
					exit(1);
				}

				ogg_stream_init(&os, ogg_page_serialno(&og));

				if (ogg_stream_pagein(&os,&og) < 0) {
					fprintf(stderr,"Error reading initial page of ogg stream.\n");
					exit(1);
				}

				if (ogg_stream_packetout(&os,&op) != 1)
				{
					fprintf(stderr,"Error reading header packet of ogg stream.\n");
					exit(1);
				}

				/* We must ensure that the ogg stream actually contains opus data */
				if (!(op.bytes > 8 && !memcmp("OpusHead", op.packet, 8)))
				{
					fprintf(stderr,"Not an ogg opus stream.\n");
					exit(1);
				}

				/* Parse the header to get stream info */
				int err = opus_head_parse(&header, op.packet, op.bytes);
				if (err)
				{
					fprintf(stderr,"Not a ogg opus stream\n");
					exit(1);
				}
				/* Now we ensure the encoding is correct for Discord */
				if (header.channel_count != 2 && header.input_sample_rate != 48000)
				{
					fprintf(stderr,"Wrong encoding for Discord, must be 48000Hz sample rate with 2 channels.\n");
					exit(1);
				}

				/* Now loop though all the pages and send the packets to the vc */
				while (ogg_sync_pageout(&oy, &og) == 1){
					ogg_stream_init(&os, ogg_page_serialno(&og));

					if(ogg_stream_pagein(&os,&og)<0){
						fprintf(stderr,"Error reading page of Ogg bitstream data.\n");
						exit(1);
					}

					while (ogg_stream_packetout(&os,&op) != 0)
					{
						/* Read remaining headers */
						if (op.bytes > 8 && !memcmp("OpusHead", op.packet, 8))
						{
							int err = opus_head_parse(&header, op.packet, op.bytes);
							if (err)
							{
								fprintf(stderr,"Not a ogg opus stream\n");
								exit(1);
							}
							if (header.channel_count != 2 && header.input_sample_rate != 48000)
							{
								fprintf(stderr,"Wrong encoding for Discord, must be 48000Hz sample rate with 2 channels.\n");
								exit(1);
							}
							continue;
						}
						/* Skip the opus tags */
						if (op.bytes > 8 && !memcmp("OpusTags", op.packet, 8))
							continue; 

						/* Send the audio */
						int samples = opus_packet_get_samples_per_frame(op.packet, 48000);

						v->voiceclient->send_audio_opus(op.packet, op.bytes, samples / 48);
					}
				}

				/* Cleanup */
				ogg_stream_clear(&os);
				ogg_sync_clear(&oy);
			}
		}
	});
	
	/* Start bot */
	bot.start(false);
	return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~~~

You can compile this example using the following command

	c++ /path/to/source.cc -ldpp -lopus -lopusfile -logg -I/usr/include/opus

\page joinvc Join or switch to the voice channel of the user issuing a command

When a user issues a command you may want to join their voice channel, e.g. in a music bot. If you are already on the same voice channel, the bot should do nothing (but be ready to instantly play audio) and if the user is on a different voice channel, the bot should switch to it. If the user is on no voice channel at all, this should be considered an error. This example shows how to do this.

\note Please be aware this example sends no audio, but indicates clearly in the comments where and how you should do so.

~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iomanip>
#include <sstream>

int main(int argc, char const *argv[])
{
	/* Setup the bot */
	dpp::cluster bot("token");

	/* Use the on_message_create event to look for commands */
	bot.on_message_create([&bot, robot, robot_size](const dpp::message_create_t & event) {

		std::stringstream ss(event.msg.content);
		std::string command;

		ss >> command;

		/* Switch to or join the vc the user is on. Syntax: .join  */
		if (command == ".join") {
			dpp::guild * g = dpp::find_guild(event.msg.guild_id);
			auto current_vc = event.from->get_voice(event.msg.guild_id);
			bool join_vc = true;
			/* Check if we are currently on any vc */
			if (current_vc) {
				/* Find the channel id  that the user is currently on */
				auto users_vc = g->voice_members.find(event.msg.author.id);
				/* See if we currently share a channel with the user */
				if (users_vc != g->voice_members.end() && current_vc->channel_id == users_vc->second.channel_id) {
					join_vc = false;
					/* We are on this voice channel, at this point we can send any audio instantly to vc:

					 * current_vc->send_audio_raw(...)
					 */
				} else {
					/* We are on a different voice channel. Leave it, then join the new one 
					 * by falling through to the join_vc branch below.
					 */
					event.from->disconnect_voice(event.msg.guild_id);
					join_vc = true;
				}
			}
			/* If we need to join a vc at all, join it here if join_vc == true */
			if (join_vc) {
				if (!g->connect_member_voice(event.msg.author.id)) {
					/* The user issuing the command is not on any voice channel, we can't do anything */
					bot.message_create(dpp::message(channel_id, "You don't seem to be on a voice channel! :("));
				} else {
					/* We are now connecting to a vc. Wait for on_voice_ready 
					 * event, and then send the audio within that event:
					 * 
					 * event.voice_client->send_audio_raw(...);
					 * 
					 * NOTE: We can't instantly send audio, as we have to wait for
					 * the connection to the voice server to be established!
					 */
				}
			}
		}
	});

	/* Start bot */
	bot.start(false);
	return 0;
}

~~~~~~~~~~~~~~~~~~~~~~~~~

\page slashcommands Using Slash Commands and Interactions

Slash commands and interactions are a newer feature of Discord which allow bot's commands to be registered centrally within the
system and for users to easily explore and get help with available commands through the client itself.

To add a slash command you should use the dpp::cluster::global_command_create method for global commands (available to all guilds)
or dpp::cluster::guild_command_create to create a local command (available only to one guild).

When a user issues these commands the reply will arrive via the on_interaction_create event which you can hook, and take action
when you see your commands. It is possible to reply to an interaction by using either the dpp::interaction_create_t::reply method,
or by manually instantiating an object of type dpp::interaction_response and attaching a dpp::message object to it.

dpp::interaction_create_t::reply has two overloaded versions of the method, one of which accepts simple std::string replies, for
basic text-only messages (if your message is 'ephemeral' you must use this) and one which accepts a dpp::message for more advanced
replies. Please note that at present, Discord only supports a small subset of message and embed features within an interaction
response object.

\note You can also use the unified command handler, which lets you combine channel based message commands and slash commands under the same lambda with the same code like they were one and the same. Note that after April of 2022 Discord will be discouraging bots from using commands that are prefixed messages via means of a privileged message intent. It is advised that you exclusively use slash commands, or the unified handler with only a prefix of "/" going forward for any new bots you create and look to migrating existing bots to this setup.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <dpp/fmt/format.h>

int main()
{
	dpp::cluster bot("token");

	/* The interaction create event is fired when someone issues your commands */
	bot.on_interaction_create([&bot](const dpp::interaction_create_t & event) {
		if (event.command.type == dpp::it_application_command) {
			dpp::command_interaction cmd_data = std::get<dpp::command_interaction>(event.command.data);
			/* Check which command they ran */
			if (cmd_data.name == "blep") {
				/* Fetch a parameter value from the command parameters */
				std::string animal = std::get<std::string>(event.get_parameter("animal"));
				/* Reply to the command. There is an overloaded version of this
				* call that accepts a dpp::message so you can send embeds.
				*/
				event.reply(dpp::ir_channel_message_with_source, fmt::format("Blep! You chose {}", animal));
			}
		}
	});

	bot.on_ready([&bot](const dpp::ready_t & event) {

		dpp::slashcommand newcommand;
		/* Create a new global command on ready event */
		newcommand.set_name("blep")
			.set_description("Send a random adorable animal photo")
			.set_application_id(bot.me.id)
			.add_option(
				dpp::command_option(dpp::co_string, "animal", "The type of animal", true).
					add_choice(dpp::command_option_choice("Dog", std::string("animal_dog"))).
					add_choice(dpp::command_option_choice("Cat", std::string("animal_cat"))).
					add_choice(dpp::command_option_choice("Penguin", std::string("animal_penguin")
				)
			)
		);

		/* Register the command */
		bot.global_command_create(newcommand);
	});

	bot.start(false);

	return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

\page spdlog Integrating with spdlog

If you want to make your bot use spdlog, like aegis does, you can attach it to the on_log event. You can do this as follows:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <spdlog/spdlog.h>
#include <spdlog/async.h>
#include <spdlog/sinks/stdout_color_sinks.h>
#include <spdlog/sinks/rotating_file_sink.h>
#include <iomanip>
#include <dpp/dpp.h>
#include <dpp/fmt/format.h>

int main(int argc, char const *argv[])
{
	dpp::cluster bot("token");

	const std::string log_name = "mybot.log";

	/* Set up spdlog logger */
	std::shared_ptr<spdlog::logger> log;
	spdlog::init_thread_pool(8192, 2);
	std::vector<spdlog::sink_ptr> sinks;
	auto stdout_sink = std::make_shared<spdlog::sinks::stdout_color_sink_mt >();
	auto rotating = std::make_shared<spdlog::sinks::rotating_file_sink_mt>(log_name, 1024 * 1024 * 5, 10);
	sinks.push_back(stdout_sink);
	sinks.push_back(rotating);
	log = std::make_shared<spdlog::async_logger>("logs", sinks.begin(), sinks.end(), spdlog::thread_pool(), spdlog::async_overflow_policy::block);
	spdlog::register_logger(log);
	log->set_pattern("%^%Y-%m-%d %H:%M:%S.%e [%L] [th#%t]%$ : %v");
	log->set_level(spdlog::level::level_enum::debug);

	/* Integrate spdlog logger to D++ log events */
	bot.on_log([&bot, &log](const dpp::log_t & event) {
		switch (event.severity) {
			case dpp::ll_trace:
				log->trace("{}", event.message);
			break;
			case dpp::ll_debug:
				log->debug("{}", event.message);
			break;
			case dpp::ll_info:
				log->info("{}", event.message);
			break;
			case dpp::ll_warning:
				log->warn("{}", event.message);
			break;
			case dpp::ll_error:
				log->error("{}", event.message);
			break;
			case dpp::ll_critical:
			default:
				log->critical("{}", event.message);
			break;
		}
	});

	/* Add the rest of your events */

	bot.start(false);
	return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

\page components Using component interactions (buttons)

Discord's newest features support sending buttons alongside messages, which when clicked by the user trigger an interaction which is routed by
D++ as an on_button_click event. To make use of this, use code as in this example.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iostream>
#include <dpp/message.h>

int main()
{
	dpp::cluster bot("token");

	/* Message handler to look for a command called !button */
	bot.on_message_create([&bot](const dpp::message_create_t & event) {
		if (event.msg.content == "!button") {
			/* Create a message containing an action row, and a button within the action row. */
			bot.message_create(
				dpp::message(event.msg.channel_id, "this text has buttons").add_component(
					dpp::component().add_component(
						dpp::component().set_label("Click me!").
						set_type(dpp::cot_button).
						set_emoji(u8"😄").
						set_style(dpp::cos_danger).
						set_id("myid")
					)
				)
			);
		}
	});

	/* When a user clicks your button, the on_button_click event will fire,
	 * containing the custom_id you defined in your button.
	 */
	bot.on_button_click([&bot](const dpp::button_click_t & event) {
		/* Button clicks are still interactions, and must be replied to in some form to
		 * prevent the "this interaction has failed" message from Discord to the user.
 		 */
		event.reply(dpp::ir_channel_message_with_source, "You clicked: " + event.custom_id);
	});

	bot.start(false);

	return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When the feature is functioning, the code below will produce buttons on the reply message like in the image below:

\image html button.png

\page commandhandler Using a command handler object

If you have many commands in your bot, and want to handle commands from multiple sources (for example modern slash commands, and more regular
prefixed channel messages) you should consider instantiating a dpp::commandhandler object. This object can be used to automatically route
commands and their parameters to functions in your program. A simple example of using this object to route commands is shown below, and will
route both the /ping (global slash command) and .ping (prefixed channel message command) to a lambda where a reply can be generated.

\note	This example automatically hooks the dpp::cluster::on_message_create and dpp::cluster::on_interaction_create events. This can be overridden if needed to allow you to still make use of these functions for your own code, if you need to do this please see the constructor documentation for dpp::commandhandler.

Note that because the dpp::commandhandler::add_command method accepts a std::function as the command handler, you may point a command handler
at a simple lambda (as shown in this example), a function pointer, or an instantiated class method of an object. This is extremely flexible
and allows you to decide how and where commands should be routed, either to an object oriented system or to a lambda based system.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>

int main()
{
	dpp::cluster bot("token");

	/* Create command handler, and specify prefixes */
	dpp::commandhandler command_handler(&bot);
	/* Specifying a prefix of "/" tells the command handler it should also expect slash commands */
	command_handler.add_prefix(".").add_prefix("/");

	bot.on_ready([&command_handler](const dpp::ready_t &event) {

		command_handler.add_command(
			/* Command name */
			"ping",

			/* Parameters */
			{
				{"testparameter", dpp::param_info(dpp::pt_string, true, "Optional test parameter") }
			},

			/* Command handler */
			[&command_handler](const std::string& command, const dpp::parameter_list_t& parameters, dpp::command_source src) {
				std::string got_param;
				if (!parameters.empty()) {
					got_param = std::get<std::string>(parameters[0].second);
				}
				command_handler.reply(dpp::message("Pong! -> " + got_param), src);
			},

			/* Command description */
			"A test ping command",

			/* Guild id (omit for a global command) */
			819556414099554344
		);

		/* NOTE: We must call this to ensure slash commands are registered.
		 * This does a bulk register, which will replace other commands
		 * that are registered already!
		 */
		command_handler.register_commands();

	});


	bot.start(false);
	return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

\page components3 Using component interactions (select menus)

This example demonstrates receiving select menu clicks and sending response messages.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>

using json = nlohmann::json;

int main()
{

	dpp::cluster bot("token");

	/* Message handler to look for a command called !select */
	bot.on_message_create([&bot](const dpp::message_create_t & event) {
		if (event.msg.content == "!select") {
			/* Create a message containing an action row, and a select menu within the action row. */
			dpp::message m(event.msg.channel_id, "this text has a select menu");
			m.add_component(
				dpp::component().add_component(
					dpp::component().set_type(dpp::cot_selectmenu).
					set_placeholder("Pick something").
					add_select_option(dpp::select_option("label1","value1","description1").set_emoji(u8"😄")).
					add_select_option(dpp::select_option("label2","value2","description2").set_emoji(u8"🙂")).
					set_id("myselid")
				)
			);
			bot.message_create(m);
		}
	});
	/* When a user clicks your select menu , the on_select_click event will fire,
	 * containing the custom_id you defined in your select menu.
	 */
	bot.on_select_click([&bot](const dpp::select_click_t & event) {
		/* Select clicks are still interactions, and must be replied to in some form to
		 * prevent the "this interaction has failed" message from Discord to the user.
		 */
		event.reply(dpp::ir_channel_message_with_source, "You clicked " + event.custom_id + " and chose: " + event.values[0]);
	});

	bot.on_log([](const dpp::log_t & event) {
		if (event.severity > dpp::ll_trace) {
			std::cout << event.message << "\n";
		}
	});

	bot.start(false);

	return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

\page components2 Using component interactions (advanced)

This example demonstrates receiving button clicks and sending response messages.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>

using json = nlohmann::json;

int main()
{

	dpp::cluster bot("token");

	bot.on_button_click([&bot](const dpp::button_click_t & event) {
		if (event.custom_id == "10") {
			event.reply(dpp::ir_channel_message_with_source, dpp::message("Correct").set_flags(dpp::m_ephemeral));
		} else {
			event.reply(dpp::ir_channel_message_with_source, dpp::message("Incorrect").set_flags(dpp::m_ephemeral));
		}
	});

	bot.on_message_create([&bot](const dpp::message_create_t & event) {
		if (event.msg.content == "!ping2") {
			bot.message_create(
				dpp::message(event.msg.channel_id, "What is 5+5?").add_component(
					dpp::component().add_component(
						dpp::component().set_label("9").
						set_style(dpp::cos_primary).
						set_id("9")
					).add_component(
						dpp::component().set_label("10").
						set_style(dpp::cos_primary).
						set_id("10")
					).add_component(
						dpp::component().set_label("11").
						set_style(dpp::cos_primary).
						set_id("11")
					)
				)
			);
		}
	});

	bot.start(false);

	return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This code will send a different message for correct and incorrect answers.

\image html button_2.png

\page subcommands Using sub-commands in slash commands

This is how to use Subcommands within your Slash Commands for your bots.

To make a subcomamnd within your command use this
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <dpp/fmt/format.h>
#include <iostream>

int main() {

	/* Setup the bot */
	dpp::cluster bot("token");
	
	/* Executes on ready. */
	bot.on_ready([&bot](const dpp::ready_t & event) {
	// Define a slash command.
	dpp::slashcommand image;
	    image.set_name("image");
	    image.set_description("Send a specific image.");
	    image.add_option(
			// Create a subcommand type option.
 	       	dpp::command_option(dpp::co_sub_command, "dog", "Send a picture of a dog.").
				add_option(dpp::command_option(dpp::co_user, "user", "User to make a dog off.", false))
			);
		image.add_option(
			// Create another subcommand type option.
	        dpp::command_option(dpp::co_sub_command, "cat", "Send a picture of a cat.").
				add_option(dpp::command_option(dpp::co_user, "user", "User to make a cat off.", false))
			);
	// Create command with a callback.
	bot.global_command_create(image, [ & ]( const dpp::confirmation_callback_t &callback ) {
	                    if(callback.is_error()) {
	                        std::cout << callback.http_info.body <<  "\n" ;
	                    }
		});
	});

	/* Use the on_interaction_create event to look for commands */
	bot.on_interaction_create([&bot](const dpp::interaction_create_t & event) {
    	    if (event.command.type == dpp::it_application_command) {
        	    dpp::command_interaction cmd_data = std::get<dpp::command_interaction>(event.command.data);

				/* Check if the command is the image command. */
				if(cmd_data.name == "image") {
					/* Check if the subcommand is "dog" */
					if(cmd_data.options[0].name == "dog") {	
						/* Checks if the subcommand has any options. */
						if(cmd_data.options[0].options.size() > 0) {
							/* Get the user option as a snowflake. */
							dpp::snowflake user = std::get<dpp::snowflake>(cmd_data.options[0].options[0].value);
							event.reply(dpp::ir_channel_message_with_source, fmt::format("<@{}> has now been turned into a dog.", user)); 
						} else {
						/* Reply if there were no options.. */
						event.reply(dpp::ir_channel_message_with_source, "<A picture of a dog.>");
						}
					}
					/* Check if the subcommand is "cat" */
					if(cmd_data.options[0].name == "cat") {
						/* Checks if the subcommand has any options. */
						if(cmd_data.options[0].options.size() > 0) {
							/* Get the user option as a snowflake. */
							dpp::snowflake user = std::get<dpp::snowflake>(cmd_data.options[0].options[0].value);
							event.reply(dpp::ir_channel_message_with_source, fmt::format("<@{}> has now been turned into a cat.", user));
						} else {
						/* Reply if there were no options.. */
						event.reply(dpp::ir_channel_message_with_source, "<A picture of a cat.>");
						}
					}
				}
			}
	});

	bot.start(false);
    return 0;
} 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

\page stream-mp3-discord-bot Streaming MP3 files

To stream MP3 files via D++ you need to link an additional dependency to your bot, namely `libmpg123`. It is relatively simple when linking this library to your bot to then decode audio to PCM and send it to the dpp::discord_voice_client::send_audio_raw function as shown below:


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <dpp/nlohmann/json.hpp>
#include <dpp/fmt/format.h>
#include <iomanip>
#include <sstream>

#include <vector>
#include <fstream>
#include <iostream>
#include <mpg123.h>
#include <out123.h>

/* For an example we will hardcode a path to some awesome music here */
#define MUSIC_FILE "/media/music/Rick Astley/Whenever You Need Somebody/Never Gonna Give You Up.mp3"

int main(int argc, char const *argv[])
{
	/* This will hold the decoded MP3.
	* The D++ library expects PCM format, which are raw sound
	* data, 2 channel stereo, 16 bit signed 48000Hz.
	*/
	std::vector<uint8_t> pcmdata;

	mpg123_init();

	int err = 0;
	unsigned char* buffer;
	size_t buffer_size, done;
	int channels, encoding;
	long rate;

	/* Note it is important to force the frequency to 48000 for Discord compatibility */
	mpg123_handle *mh = mpg123_new(NULL, &err);
	mpg123_param(mh, MPG123_FORCE_RATE, 48000, 48000.0);

	/* Decode entire file into a vector. You could do this on the fly, but if you do that
	* you may get timing issues if your CPU is busy at the time and you are streaming to
	* a lot of channels/guilds.
	*/
	buffer_size = mpg123_outblock(mh);
	buffer = new unsigned char[buffer_size];

	/* Note: In a real world bot, this should have some error logging */
	mpg123_open(mh, MUSIC_FILE);
	mpg123_getformat(mh, &rate, &channels, &encoding);

	unsigned int counter = 0;
	for (int totalBtyes = 0; mpg123_read(mh, buffer, buffer_size, &done) == MPG123_OK; ) {
		for (auto i = 0; i < buffer_size; i++) {
			pcmdata.push_back(buffer[i]);
		}
		counter += buffer_size;
		totalBtyes += done;
	}
	delete buffer;
	mpg123_close(mh);
	mpg123_delete(mh);

	/* Setup the bot */
	dpp::cluster bot("token");

	/* Use the on_message_create event to look for commands */
	bot.on_message_create([&bot, &pcmdata](const dpp::message_create_t & event) {
		std::stringstream ss(event.msg.content);
		std::string command;
		ss >> command;

		/* Tell the bot to join the discord voice channel the user is on. Syntax: .join */
		if (command == ".join") {
			dpp::guild * g = dpp::find_guild(event.msg.guild_id);
			if (!g->connect_member_voice(event.msg.author.id)) {
				bot.message_create(dpp::message(event.msg.channel_id, "You don't seem to be on a voice channel! :("));
			}
		}

		/* Tell the bot to play the mp3 file. Syntax: .mp3 */
		if (command == ".mp3") {
			dpp::voiceconn* v = event.from->get_voice(event.msg.guild_id);
			if (v && v->voiceclient && v->voiceclient->is_ready()) {
				/* Stream the already decoded MP3 file. This passes the PCM data to the library to be encoded to OPUS */
				v->voiceclient->send_audio_raw((uint16_t*)pcmdata.data(), pcmdata.size());
			}
		}
	});

	/* A basic logger */
	bot.on_log([](const dpp::log_t & event) {
		if (event.severity > dpp::ll_trace) {
			std::cout << event.message << "\n";
		}
	});

	/* Start bot */
	bot.start(false);

	/* Clean up */
	mpg123_exit();

	return 0;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To compile this program you must remember to specify `libmpg123` alongside `libdpp` in the build command, for example:

` g++ -std=c++17 -o musictest musictest.cpp -lmpg123 -ldpp`

\page record-user Record yourself in a VC

DPP supports receiving audio. This examples show how to use it to record some user in a VC.

\note Voice receiving by bots is not officially supported by the Discord API. We cannot guarantee that this feature will work in the future. 

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iomanip>
#include <sstream>

int main(int argc, char const *argv[])
{
	/* Example to record a user in a VC
	* 
	* Recording is output as './me.pcm' and you can play it via the soundboard example
	* or use ffmpeg 'ffplay -f s16le -ar 48000 -ac 2 -i ./me.pcm'
	*/

	/* Replace with the user's id you wish to record */
	dpp::snowflake user_id = 407877550216314882;

	dpp::cluster bot("token");

	FILE *fd;
	fd = fopen("./me.pcm", "wb");

	/* Use the on_message_create event to look for commands */
	bot.on_message_create([&bot, &fd](const dpp::message_create_t & event) {

		std::stringstream ss(event.msg.content);
		std::string command;

		ss >> command;

		/* Tell the bot to record */
		if (command == ".record") {
			dpp::guild * g = dpp::find_guild(event.msg.guild_id);
			if (!g->connect_member_voice(event.msg.author.id)) {
				bot.message_create(dpp::message(
					event.msg.channel_id, 
					"You don't seem to be on a voice channel! :("
				));
			}
		}

		/* Tell the bot to stop recording */
		if (command == ".stop") {
			event.from->disconnect_voice(event.msg.guild_id);
			fclose(fd);
		}
	});

	bot.on_voice_receive([&bot, &fd, &user_id](const dpp::voice_receive_t &event) {
		if (event.user_id == user_id) {
			fwrite((char *)event.audio, 1, event.audio_size, fd);
		}
	});

	/* Start bot */
	bot.start(false);
	return 0;
}
~~~~~~~~~~

\page embed-message Sending Embeds

You might have seen these special messages, often sent by bots. In this section, we will show how to create an embed.

To make an embed use this.

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>

int main() {
    /* Setup the bot */
    dpp::cluster bot("token");

    /* Message handler to look for a command called !embed */
    bot.on_message_create([&bot](const dpp::message_create_t & event) {
        if (event.msg.content == "!embed") {

            /* create the embed */
            dpp::embed embed = dpp::embed().
                set_color(0x0099ff).
                set_title("Some name").
                set_url("https://dpp.dev/").
                set_author("Some name", "https://dpp.dev/", "https://dpp.dev/DPP-Logo.png").
                set_description("Some description here").
                set_thumbnail("https://dpp.dev/DPP-Logo.png").
                add_field(
                        "Regular field title",
                        "Some value here"
                ).
                add_field(
                        "Inline field title",
                        "Some value here",
                        true
                ).
                add_field(
                        "Inline field title",
                        "Some value here",
                        true
                ).
                set_image("https://dpp.dev/DPP-Logo.png").
                set_footer(dpp::embed_footer().set_text("Some footer text here").set_icon("https://dpp.dev/DPP-Logo.png")).
                set_timestamp(time(0));

            /* reply with the created embed */
            bot.message_create(dpp::message(event.msg.channel_id, embed).set_reference(event.msg.id));
        }
    });

    bot.start(false);
    return 0;
}
~~~~~~~~~~

The code will send the following message.

\image html embed.png

\page application-command-autocomplete Slash command auto completion

Discord now supports sending auto completion lists for slash command choices. To use this feature you can use code such as the example below:

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
 
int main()
{
	dpp::cluster bot("token");

	bot.on_ready([&bot](const dpp::ready_t & event) {
		/* Create a new global command on ready event */
		bot.global_command_create(dpp::slashcommand().set_name("blep")
			.set_description("Send a random adorable animal photo")
			.set_application_id(bot.me.id)
			.add_option(
				/* If you set the auto complete setting on a command option, it will trigger the on_auticomplete
				 * event whenever discord needs to fill information for the choices. You cannot set any choices
				 * here if you set the auto complete value to true.
				 */
				dpp::command_option(dpp::co_string, "animal", "The type of animal").set_auto_complete(true)
			)
		);
	});

	/* The interaction create event is fired when someone issues your commands */
	bot.on_interaction_create([&bot](const dpp::interaction_create_t & event) {
		if (event.command.type == dpp::it_application_command) {
			dpp::command_interaction cmd_data = std::get<dpp::command_interaction>(event.command.data);
			/* Check which command they ran */
			if (cmd_data.name == "blep") {
				/* Fetch a parameter value from the command parameters */
				std::string animal = std::get<std::string>(event.get_parameter("animal"));
				/* Reply to the command. There is an overloaded version of this
				* call that accepts a dpp::message so you can send embeds.
				*/
				event.reply(dpp::ir_channel_message_with_source, "Blep! You chose " + animal);
			}
		}
	});
 
	/* The on_autocomplete event is fired whenever discord needs information to fill in a command options's choices.
	 * You must reply with a REST event within 500ms, so make it snappy!
	 */
	bot.on_autocomplete([&bot](const dpp::autocomplete_t & event) {
		for (auto & opt : event.options) {
			/* The option which has focused set to true is the one the user is typing in */
			if (opt.focused) {
				/* In a real world usage of this function you should return values that loosely match
				 * opt.value, which contains what the user has typed so far. The opt.value is a variant
				 * and will contain the type identical to that of the slash command parameter.
				 * Here we can safely know it is string.
				 */
				std::string uservalue = std::get<std::string>(opt.value);
				bot.interaction_response_create(event.command.id, event.command.token, dpp::interaction_response(dpp::ir_autocomplete_reply)
					.add_autocomplete_choice(dpp::command_option_choice("squids", "lots of squids"))
					.add_autocomplete_choice(dpp::command_option_choice("cats", "a few cats"))
					.add_autocomplete_choice(dpp::command_option_choice("dogs", "bucket of dogs"))
					.add_autocomplete_choice(dpp::command_option_choice("elephants", "bottle of elephants"))
				);
				bot.log(dpp::ll_debug, "Autocomplete " + opt.name + " with value '" + uservalue + "' in field " + event.name);
				break;
			}
		}
	});

	/* Simple log event */
	bot.on_log([&bot](const dpp::log_t & event) {
		std::cout << dpp::utility::loglevel(event.severity) << ": " << event.message << "\n";
	});

	bot.start(false);

	return 0;
}
~~~~~~~~~~

\page attach-file Attaching a file to a message

Attached files must be locally stored.

To attach a file to a message, you can upload a local image.

D++ has this helper function to read a file: `dpp::utility::read_file`.

An example program:

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>

int main() {
    dpp::cluster bot("token");

    /* Message handler to look for a command called !file */
    bot.on_message_create([&bot](const dpp::message_create_t &event) {
        if (event.msg.content == "!file") {
            // create a message
            dpp::message msg(event.msg.channel_id, "Hey there, i've got a new file!");

            // attach the file
            msg.set_file_content(dpp::utility::read_file("path_to_your_file.txt"));
            
            /*
             * alternatively, you can put any other name in here.
             * This name doesn't have to be the same as the uploaded filename.
             * But it should have the same file extension.
             */
            msg.set_filename("file.txt");

            // send the message
            bot.message_create(msg);
        }
    });

    bot.start(false);
    return 0;
}
~~~~~~~~~~

Attachments via an url aren't possible. But there's a workaround for. You can download the file and then attach it to the message.

To make requests, D++ also has a helper function: `dpp::cluster::request`.

The following example program shows how to request a file and attach it to a message.

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>

int main() {
    dpp::cluster bot("token");

    /* Message handler to look for a command called !file */
    bot.on_message_create([&bot](const dpp::message_create_t &event) {
        if (event.msg.content == "!file") {
            // request an image
            bot.request("https://dpp.dev/DPP-Logo.png", dpp::m_get, [&bot, channel_id = event.msg.channel_id](const dpp::http_request_completion_t & httpRequestCompletion) {

                // create a message
                dpp::message msg(channel_id, "This is my new attachment:");

                // attach the image on success
                if (httpRequestCompletion.status == 200) {
                    msg.set_file_content(httpRequestCompletion.body);
                    msg.set_filename("example-image.png"); // give the file a name
                }

                // send the message
                bot.message_create(msg);
            });
        }
    });

    bot.start(false);
    return 0;
}
~~~~~~~~~~

Here's another example of how to add a local image to an embed.

Upload the image in the same message as the embed and then reference it in the embed.

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>

int main() {
    dpp::cluster bot("token");

    /* Message handler to look for a command called !file */
    bot.on_message_create([&bot](const dpp::message_create_t &event) {
        if (event.msg.content == "!file") {
            // create a message
            dpp::message msg(event.msg.channel_id, "");

            // attach the file to the message
            msg.set_file_content(dpp::utility::read_file("path_to_your_image.jpg"));
            msg.set_filename("image.jpg");

            dpp::embed embed;
            embed.set_image("attachment://image.jpg"); // reference to the attached file
            msg.add_embed(embed);

            // send the message
            bot.message_create(msg);
        }
    });

    bot.start(false);
    return 0;
}
~~~~~~~~~~

\page caching-messages Caching Messages

By default D++ does not cache messages. The example program below demonstrates how to instantiate a custom cache using dpp::cache which will allow you to cache messages and query the cache for messages by ID.

This can be adjusted to cache any type derived from dpp::managed including types you define yourself.

@note This example will cache and hold onto messages forever! In a real world situation this would be bad. If you do use this,
you should use the dpp::cache::remove() method periodically to remove stale items. This is left out of this example as a learning
exercise to the reader. For further reading please see the documentation of dpp::cache

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <sstream>

int main() {
	/* Create bot */
	dpp::cluster bot("token");

	/* Create a cache to contain types of dpp::message */
	dpp::cache<dpp::message> message_cache;

	/* Message handler */
	bot.on_message_create([&](const dpp::message_create_t &event) {

		/* Make a permanent pointer using new, for each message to be cached */
		dpp::message* m = new dpp::message();
		/* Store the message into the pointer by copying it */
		*m = event.msg;
		/* Store the new pointer to the cache using the store() method */
		message_cache.store(m);

		/* Simple ghetto command handler. In the real world, use slashcommand or commandhandler here. */
		std::stringstream ss(event.msg.content);
		std::string cmd;
		dpp::snowflake msg_id;
		ss >> cmd;

		/* Look for our command */
		if (cmd == "!get") {
			ss >> msg_id;
			/* Search our cache for a cached message */
			dpp::message* find_msg = message_cache.find(msg_id);
			if (find_msg != nullptr) {
				/* Found a cached message, echo it out */
				bot.message_create(dpp::message(event.msg.channel_id, "This message had the following content: " + find_msg->content));
			} else {
				/* Nothing like that here. Have you checked under the carpet? */
				bot.message_create(dpp::message(event.msg.channel_id, "There is no message cached with this ID"));
			}
		}
	});

	/* Start bot */
	bot.start(false);

	return 0;
}
~~~~~~~~~~

\page collecting-reactions Collecting Reactions

D++ comes with many useful helper classes, but amongst these is something called dpp::collector. Collector is a template which can be specialised to automatically collect objects of a pre-determined type from events for a specific interval of time. Once this time period is up, or the class is otherwise signalled, a method is called with the complete set of collected objects.

In the example below we will use it to collect all reactions on a message.

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>

/* To create a collector we must derive from dpp::collector. As dpp::collector is a complicated template,
 * various pre-made forms exist such as this one, reaction_collector.
 */
class react_collector : public dpp::reaction_collector {
public:
	/* Collector will run for 20 seconds */
	react_collector(dpp::cluster* cl, snowflake id) : dpp::message_collector(cl, 20, id) { }

	/* On completion just output number of collected reactions to as a message. */
	virtual void completed(const std::vector<dpp::collected_reaction>& list) {
		if (list.size()) {
			owner->message_create(dpp::message(list[0].channel_id, "I collected " + std::to_string(list.size()) + " reactions!"));
		} else {
			owner->message_create(dpp::message("... I got nothin'."));
		}
	}
};


int main() {
	/* Create bot */
	dpp::cluster bot("token");

	/* Pointer to reaction collector */
	react_collector* r = nullptr;

	/* Message handler */
	bot.on_message_create([&](const dpp::message_create_t &event) {

		/* If someone sends a message that has the text 'collect reactions!' start a reaction collector */
		if (event.msg.content == "collect reactions!" && r == nullptr) {
			/* Create a new reaction collector to collect reactions */
			r = new react_collector(&bot, event.msg.id);
		}

	});

	/* Start bot */
	bot.start(false);

	return 0;
}
~~~~~~~~~~

\page modal-dialog-interactions Modal Dialog Interactions

Modal dialog interactions are a new Discord API feature that allow you to have pop-up windows which prompt the user to input information. Once the user has filled in this information, your program will receive an `on_form_submit` event which will contain the data which was input. You must use a slash command interaction response to submit your modal form data to Discord, via the `on_interaction_create` event. From here calling the `dialog` method of the `interaction_create_t` event object will trigger the dialog to appear.

@warning This feature is currently in a closed beta, and requires whitelisting of your application by Discord. These documents and any library methods associated with this feature are subject to change until the feature is officially announced for general use!

Each dialog box may have up to five rows of input fields. The example below demonstrates a simple setup with just one text input:

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iostream>

int main(int argc, char const *argv[])
{
	dpp::cluster bot("token");

	bot.on_ready([&](const dpp::ready_t & event) {
		/* Create a slash command and register it as a global command */
		dpp::slashcommand newcommand;
		newcommand.set_name("dialog").set_description("Make a modal dialog box").set_application_id(bot.me.id);
		bot.global_command_create(newcommand);
	});

	bot.on_interaction_create([&bot](const dpp::interaction_create_t & event) {
		if (event.command.type == dpp::it_application_command) {
			dpp::command_interaction cmd_data = std::get<dpp::command_interaction>(event.command.data);
			/* Check for our /dialog command */
			if (cmd_data.name == "dialog") {
				/* Instantiate an interaction_modal_response object */
				dpp::interaction_modal_response modal("my_modal", "Please enter stuff");
				/* Add a text component */
				modal.add_component(
					dpp::component().
					set_label("Type rammel").
					set_id("field_id").
					set_type(dpp::cot_text).
					set_placeholder("gumf").
					set_min_length(1).
					set_max_length(2000).
					set_text_style(dpp::text_paragraph)
				);
				/* Trigger the dialog box. All dialog boxes are ephemeral */
				event.dialog(modal);
			}
		}
	});

	/* This event handles form submission for the modal dialog we create above */
	bot.on_form_submit([&](const dpp::form_submit_t & event) {
		/* For this simple example we know the first element of the first row ([0][0]) is value type string.
		 * In the real world it may not be safe to make such assumptions!
		 */
		std::string v = std::get<std::string>(event.components[0].components[0].value);
		dpp::message m;
		m.set_content("You entered: " + v).set_flags(dpp::m_ephemeral);
		/* Emit a reply. Form submission is still an interaction and must generate some form of reply! */
		event.reply(dpp::ir_channel_message_with_source, m);
	});

	/* Budget brand logger */
	bot.on_log([&](const dpp::log_t & log) {
		std::cout << log.message << "\n";
	});

	/* Start bot */
	bot.start(false);
	return 0;
}
~~~~~~~~~~

If you compile and run this program and wait for the global command to register, typing `/dialog` will present you with a dialog box like the one below:

\image html modal_dialog.png

\page context-menu Context Menus

Context menus are application commands that appear on the context menu (right click or tap) of users or messages to perform context-specific actions. They can be created using `dpp::slashcommand`. Once you create a context menu, try right-clicking either a user or message to see it in your server!

\image html context_menu_user_command.png

The following example shows how to create and handle **user context menus**.

~~~~~~~~~~{.cpp}
#include <dpp/dpp.h>
#include <iostream>

int main()
{
    dpp::cluster bot("token");

    bot.on_ready([&bot](const dpp::ready_t &event) {
        dpp::slashcommand command;
        /* Define a slash command */
        command.set_name("High Five")
                .set_type(dpp::ctxm_user)
                .set_application_id(bot.me.id);
        /* Register the command */
        bot.guild_command_create(command, 857692897221033129); // you need to put your guild-id in here
    });

    /* Use the on_interaction_create event to look for application commands */
    bot.on_interaction_create([&](const dpp::interaction_create_t &event) {
        if (event.command.type == dpp::it_application_command) {
            dpp::command_interaction cmd_data = std::get<dpp::command_interaction>(event.command.data);
            
            /* check if the command is a user context menu action */
            if (cmd_data.type == dpp::ctxm_user) {

                /* check if the context menu name is High Five */
                if (cmd_data.name == "High Five") {
                    dpp::user user = event.command.resolved.users.begin()->second; // the user who the command has been issued on
                    dpp::user author = event.command.usr; // the user who clicked on the context menu
                    event.reply(dpp::ir_channel_message_with_source, author.get_mention() + " slapped " + user.get_mention());
                }
            }
        }
    });

    /* Start bot */
    bot.start(false);

    return 0;
}
~~~~~~~~~~

It registers a guild command that can be called by right-click a user and click on the created menu.

\image html context_menu_user_command_showcase.png
